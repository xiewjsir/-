##### 环境介绍：
```
系统版本：Centos7.6
内核：3.10.0-957.12.1.el7.x86_64
Kubernetes:v1.14.1
Docker-ce:18.09.6 #kubeadm 目前只支持到这个版本  

Keepalived保证apiserever服务器的IP高可用
Haproxy实现apiserver的负载均衡
master x2 && etcd x2 保证k8s集群可用性

192.168.1.66        master+etcd2
192.168.1.69        master+etcd1
192.168.1.64        Keepalived + Haproxy1
192.168.1.70        Keepalived + Haproxy2
192.168.1.67        node1
192.168.1.68        node2
192.168.1.88        VIP、apiserver的地址
```
##### 一、准备工作
>为方便操作，所有操作均以root用户执行 以下操作仅在kubernetes集群节点执行即可

- 关闭selinux和防火墙
```
sed -ri 's#(SELINUX=).*#\1disabled#' /etc/selinux/config
setenforce 0

systemctl disable firewalld
systemctl stop firewalld
```
- 关闭swap
```
swapoff -a
```

- 配置转发相关参数，否则可能会出错
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF

sysctl --system
```
- 加载ipvs模块
```
cat << EOF > /etc/sysconfig/modules/ipvs.modules 
#!/bin/bash
ipvs_modules_dir="/usr/lib/modules/\`uname -r\`/kernel/net/netfilter/ipvs"
for i in \`ls \$ipvs_modules_dir | sed  -r 's#(.*).ko.*#\1#'\`; do
    /sbin/modinfo -F filename \$i  &> /dev/null
    if [ \$? -eq 0 ]; then
        /sbin/modprobe \$i
    fi
done
EOF

chmod +x /etc/sysconfig/modules/ipvs.modules 
bash /etc/sysconfig/modules/ipvs.modules
```
- 安装cfssl
```
#在master节点安装即可！！！

wget -O /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget -O /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget -O /bin/cfssl-certinfo  https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
for cfssl in `ls /bin/cfssl*`;do chmod +x $cfssl;done;
```
- 安装kubernetes阿里云镜像
```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y  kubelet kubeadm kubectl
```
- 安装docker，并干掉docker0网桥
```
#删除旧docker
yum list install|grep docker
yum remove ***

wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install -y docker-ce

mkdir /etc/docker/
cat << EOF > /etc/docker/daemon.json
{   "registry-mirrors": ["https://registry.docker-cn.com"],
    "live-restore": true,
    "default-shm-size": "128M",
    "bridge": "none",
    "max-concurrent-downloads": 10,
    "oom-score-adjust": -1000,
    "debug": false
}	
EOF	

#重启docker
systemctl daemon-reload
systemctl enable docker
systemctl restart docker
```

- 配置hosts文件
```
192.168.1.64        master1
192.168.1.64        etcd1
192.168.1.66        master2
192.168.1.66        etcd2
192.168.1.69        lb1
192.168.1.70        lb2
192.168.1.67        node1
192.168.1.68        node2
```

##### 二、配置etcd
- 配置etcd的证书
```
mkdir -pv $HOME/ssl && cd $HOME/ssl
	
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF


cat > etcd-ca-csr.json << EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shenzhen",
      "L": "Shenzhen",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ]
}
EOF


cat > etcd-csr.json << EOF
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "192.168.1.69",
      "192.168.1.66"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shenzhen",
            "L": "Shenzhen",
            "O": "etcd",
            "OU": "Etcd Security"
        }
    ]
}
EOF

#生成证书并复制证书至其他etcd节点

cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca
cfssl gencert -ca=etcd-ca.pem -ca-key=etcd-ca-key.pem -config=ca-config.json -profile=kubernetes etcd-csr.json | cfssljson -bare etcd

mkdir -pv /etc/etcd/ssl
mkdir -pv /etc/kubernetes/pki/etcd
cp etcd*.pem /etc/etcd/ssl
cp etcd*.pem /etc/kubernetes/pki/etcd

scp -r /etc/etcd 192.168.1.69:/etc/
scp -r /etc/etcd 192.168.1.66:/etc/

```

- etcd1主机启动etcd
```
yum install -y etcd 
		
cat << EOF > /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="https://192.168.1.69:2380"
ETCD_LISTEN_CLIENT_URLS="https://127.0.0.1:2379,https://192.168.1.69:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="etcd1"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.69:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://127.0.0.1:2379,https://192.168.1.69:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.1.69:2380,etcd2=https://192.168.1.66:2380"
ETCD_INITIAL_CLUSTER_TOKEN="BigBoss"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[Security]
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
#ETCD_CLIENT_CERT_AUTH="false"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
#ETCD_AUTO_TLS="false"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
#ETCD_PEER_CLIENT_CERT_AUTH="false"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
#ETCD_PEER_AUTO_TLS="false"
#
#[Logging]
#ETCD_DEBUG="false"
#ETCD_LOG_PACKAGE_LEVELS=""
#ETCD_LOG_OUTPUT="default"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#ETCD_AUTO_COMPACTION_RETENTION="0"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#ETCD_METRICS="basic"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple"
EOF

chown -R etcd.etcd /etc/etcd
systemctl enable etcd
systemctl start etcd
```
- etcd2主机启动etcd
```
yum install -y etcd 
		
cat << EOF > /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="https://192.168.1.66:2380"
ETCD_LISTEN_CLIENT_URLS="https://127.0.0.1:2379,https://192.168.1.66:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="etcd2"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.66:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://127.0.0.1:2379,https://192.168.1.66:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.1.69:2380,etcd2=https://192.168.1.66:2380"
ETCD_INITIAL_CLUSTER_TOKEN="BigBoss"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[Security]
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
#ETCD_CLIENT_CERT_AUTH="false"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
#ETCD_AUTO_TLS="false"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
#ETCD_PEER_CLIENT_CERT_AUTH="false"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
#ETCD_PEER_AUTO_TLS="false"
#
#[Logging]
#ETCD_DEBUG="false"
#ETCD_LOG_PACKAGE_LEVELS=""
#ETCD_LOG_OUTPUT="default"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#ETCD_AUTO_COMPACTION_RETENTION="0"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#ETCD_METRICS="basic"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple"
EOF

chown -R etcd.etcd /etc/etcd
systemctl enable etcd
systemctl start etcd
```
- 检查etcd集群
```
etcdctl --endpoints "https://192.168.1.69:2379,https://192.168.1.66:2379"   --ca-file=/etc/etcd/ssl/etcd-ca.pem  \
--cert-file=/etc/etcd/ssl/etcd.pem   --key-file=/etc/etcd/ssl/etcd-key.pem   cluster-health


[root@node3 ~]# etcdctl --endpoints "https://192.168.1.69:2379,https://192.168.1.66:2379"   --ca-file=/etc/etcd/ssl/etcd-ca.pem  \
> --cert-file=/etc/etcd/ssl/etcd.pem   --key-file=/etc/etcd/ssl/etcd-key.pem   cluster-health
member 3639deb1869a1bda is healthy: got healthy result from https://127.0.0.1:2379
member b75e13f1faa57bd8 is healthy: got healthy result from https://127.0.0.1:2379
member e31fec5bb4c882f2 is healthy: got healthy result from https://127.0.0.1:2379
```
##### 配置keepalived
- 在lb1机器上配置
```
yum install -y keepalived

cat << EOF > /etc/keepalived/keepalived.conf
! Configuration File for keepalived
    global_defs {
        notification_email {
            root@localhost      #发送邮箱
        }
        notification_email_from keepalived@localhost    #邮箱地址   
        smtp_server 127.0.0.1   #邮件服务器地址
        smtp_connect_timeout 30 
        router_id node1         #主机名，每个节点不同即可
        vrrp_mcast_group4 224.0.100.100    #组播地址
    }       
        
vrrp_instance VI_1 {
    state MASTER        #在另一个节点上为BACKUP
    interface eth0      #IP地址漂移到的网卡
    virtual_router_id 6 #多个节点必须相同
    priority 100        #优先级，备用节点的值必须低于主节点的值
    advert_int 1        #通告间隔1秒
    authentication {
        auth_type PASS      #预共享密钥认证
        auth_pass 571f97b2  #密钥
    }
    virtual_ipaddress {
        192.168.1.88/24    #VIP地址
    }
}	
EOF

systemctl enable keepalived
systemctl start keepalived
```
- 在lb2主机配置
```
yum install -y keepalived

cat << EOF > /etc/keepalived/keepalived.conf
! Configuration File for keepalived
    global_defs {
        notification_email {
            root@localhost      #发送邮箱
        }
        notification_email_from keepalived@localhost    #邮箱地址   
        smtp_server 127.0.0.1   #邮件服务器地址
        smtp_connect_timeout 30 
        router_id node2         #主机名，每个节点不同即可
        vrrp_mcast_group4 224.0.100.100  #组播地址
    }       
        
vrrp_instance VI_1 {
    state BACKUP        #在另一个节点上为MASTER
    interface eth0      #IP地址漂移到的网卡
    virtual_router_id 6 #多个节点必须相同
    priority 80        #优先级，备用节点的值必须低于主节点的值
    advert_int 1        #通告间隔1秒
    authentication {
        auth_type PASS      #预共享密钥认证
        auth_pass 571f97b2  #密钥
    }
    virtual_ipaddress {
        192.168.1.88/24    #漂移过来的IP地址
    }
}	
EOF

systemctl enable keepalived
systemctl start keepalived
```

##### 配置Haproxy
- 在lb1主机上
```
yum install  -y haproxy

cat << EOF > /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

defaults
    mode                    tcp
    log                     global
    retries                 3
    timeout connect         10s
    timeout client          1m
    timeout server          1m

frontend kubernetes
    bind *:6443
    mode tcp
    default_backend kubernetes-master

backend kubernetes-master
    balance roundrobin
    server master1  192.168.1.64:6443 check maxconn 2000
    server master2  192.168.1.70:6443 check maxconn 2000
EOF

systemctl enable haproxy
systemctl start haproxy
```
- 在lb2主机上
```
yum install  -y haproxy

cat << EOF > /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

defaults
    mode                    tcp
    log                     global
    retries                 3
    timeout connect         10s
    timeout client          1m
    timeout server          1m

frontend kubernetes
    bind *:6443
    mode tcp
    default_backend kubernetes-master

backend kubernetes-master
    balance roundrobin
    server master1  192.168.1.64:6443 check maxconn 2000
    server master2  192.168.1.70:6443 check maxconn 2000
EOF

systemctl enable haproxy
systemctl start haproxy
```

##### 初始化master
- 初始化master1
```
#kubeadm init配置文件参考：https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file

cd $HOME
cat << EOF > /root/kubeadm-init.yaml
apiVersion: kubeadm.k8s.io/v1alpha3
kind: InitConfiguration
apiEndpoint:
  advertiseAddress: 192.168.1.69 
  bindPort: 6443
---
apiVersion: kubeadm.k8s.io/v1alpha3
kind: ClusterConfiguration  
apiServerCertSANs:           	#此处填所有的masterip和lbip和其它你可能需要通过它访问apiserver的地址和域名或者主机名等
- master1
- master2
- 192.168.1.64
- 192.168.1.66
- 192.168.1.69
- 192.168.1.70
- 192.168.1.88
- 127.0.0.1
etcd:    #ETCD的地址
  external:
    endpoints:
    - "https://192.168.1.69:2379"
    - "https://192.168.1.66:2379"
    caFile: /etc/kubernetes/pki/etcd/etcd-ca.pem
    certFile: /etc/kubernetes/pki/etcd/etcd.pem
    keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem
networking:
  podSubnet: 10.244.0.0/16    	# pod网络的网段
kubernetesVersion: "v1.14.1"    # kubernetes的版本 
controlPlaneEndpoint: 192.168.1.88:6443   #VIP地址  
featureGates:
  CoreDNS: true
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers  # image的仓库源
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
EOF

systemctl enable kubelet
kubeadm config migrate --old-config kubeadm-init.yaml --new-config kubeadm-init.yaml
kubeadm config images pull --config kubeadm-init.yaml
docker tag  registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1  k8s.gcr.io/pause:3.1

kubeadm init --config /root/kubeadm-init.yaml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

cat << EOF > /etc/profile.d/kubernetes.sh 
source <(kubectl completion bash)
EOF
source /etc/profile.d/kubernetes.sh 

scp -r /etc/kubernetes/pki 192.168.1.66/etc/kubernetes/
```