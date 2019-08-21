##### 安装
```
docker volume create --name=sentry-data && docker volume create --name=sentry-postgres
cp -n .env.example .env

vim config.yml
###############
# Mail Server #
###############

# mail.backend: 'smtp'  # Use dummy if you want to disable email entirely
mail.host: 'smtp.qq.com'
mail.port: 25
mail.username: '183778760@qq.com'
mail.password: '授权码'
mail.use-tls: false
# The email address to send on behalf of
mail.from: '183778760@qq.com'

###################
# System Settings #
###################

# If this file ever becomes compromised, it's important to regenerate your a new key
# Changing this value will result in all current sessions being invalidated.
# A new key can be generated with `$ sentry config generate-secret-key`
# system.secret-key: 'changeme'
system.url-prefix: 'http://ip:9000'

#修改docker-compose.yml文件，增加
privileged: true

docker-compose build
docker-compose run --rm web config generate-secret-key #将生成的密钥添加到.env
docker-compose run --rm web upgrade
docker-compose up -d
localhost:9000

#Settings->sentry->laravel(项目名)->Client Keys  获取DSN
```

##### 使用
- larave5.x
```
composer require sentry/sentry-laravel:1.0.2

#config/app.php
'providers' => array(
    // ...
    Sentry\Laravel\ServiceProvider::class,
)

'aliases' => array(
    // ...
    'Sentry' => Sentry\Laravel\Facade::class,
)

#App/Exceptions/Handler.php
public function report(Exception $exception)
{
    if (app()->bound('sentry') && $this->shouldReport($exception)){
        app('sentry')->captureException($exception);
    }

    parent::report($exception);
}

php artisan vendor:publish --provider="Sentry\Laravel\ServiceProvider"

#触发异常
Route::get('/debug-sentry', function () {
    throw new Exception('My first Sentry error!');
});
```

##### 问题汇总
- 邀请邮件过期，系统时间问题
```
date +"%Y-%m-%d %H:%M:%S"
date -s YY/MM/DD
date -s HH:MM::SS

tzselect
#以上操作无效

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 邮件发送规则配置：Settings->sentry->laravel->Alerts (并不是异常抛出时就发邮件通知，而是间隔多长时间发送一次)
    - An event is seen ： 一个事件发生的时候
    - An issue is first seen ： 第一个发生错误的时候
    - An issue changes state from resolved to unresolved ：问题从解决到未解决的时候
    - An event's tags match {key} {match} {value}  ： 匹配到 tags 的键值对的时候发送
    - An issue is seen more than {value} times in {interval} ：在固定时间内出现次数匹配的时候
    - An issue is seen by more than {value} users in {interval} ：在固定时间内出现用户的次数匹配的时候
    - An event's {attribute} value {match} {value} ： 匹配到某一个事件的时候
    - An event's level is {match} {level} ： 事件级别匹配的时候

##### 参考文档
- [官方git](https://github.com/getsentry/sentry)
- [DSN为空解决方法](https://github.com/getsentry/onpremise/issues/184)
- [容器中root用户没有修改时间权限问题](https://segmentfault.com/q/1010000004670542/a-1020000004672753)
- [date命令修改Linux系统的时间无效](http://www.cleey.com/blog/single/id/862.html)
- [配置钉钉和邮箱](https://www.cnblogs.com/elsons/p/10986036.html)