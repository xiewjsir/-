###### grant_type:
- client_credentials    凭证式
- authorization_code    授权码
- password  密码式


```
//authorization_code
//拿到code,附加在redirect_uri，并跳转到redirect_uri
http://localhost:9096/authorize?client_id=1&redirect_uri=http://localhost:9094/oauth2&response_type=code&scope=all&state=xyz

//redirect_uri 端拿到返回的code信息，组装以下链接，拿到token
http://localhost:9096/token?client_id=1&client_secret=123456&code=UCGZNL5EPMC-LVA8UG4YEQ&grant_type=authorization_code&redirect_uri=http%3A%2F%2Flocalhost%3A9094%2Foauth2

//client_credentials
http://localhost:9096/token?grant_type=client_credentials&client_id=1&client_secret=123456&scope=read

//password
http://localhost:9096/token?grant_type=password&username=wayne&password=zhidekan2202&scope=read&client_id=1&client_secret=123456

//通过token访问被保护资源，也可通过 Authorization 方式
http://localhost:9096/test?access_token=xxx

```

```
package main

import (
	"encoding/json"
	"fmt"
	"github.com/dgrijalva/jwt-go"
	"github.com/go-session/session"
	_ "github.com/go-sql-driver/mysql"
	"gopkg.in/go-oauth2/mysql.v3"
	"gopkg.in/oauth2.v3/errors"
	"gopkg.in/oauth2.v3/generates"
	"gopkg.in/oauth2.v3/manage"
	"gopkg.in/oauth2.v3/server"
	"gopkg.in/oauth2.v3/store"
	"log"
	"net/http"
	"net/url"
	"omc/oauth-server/db"
	"os"
	"time"
	"omc/oauth-server/models"
)

func main() {
	//创建Manager实例
	manager := manage.NewDefaultManager()
	//设置授权码模式令牌的配置参数
	manager.SetAuthorizeCodeTokenCfg(manage.DefaultAuthorizeCodeTokenCfg)
	//强制映射访问令牌存储接口
	manager.MustTokenStorage(store.NewMemoryTokenStore())

	//映射令牌存储接口
	manager.MapTokenStorage(
		mysql.NewStore(mysql.NewConfig("root:root@tcp(192.168.1.66:3306)/zdk_iot?charset=utf8"), "", 0),
	)

	clientStore := db.NewDefaultStore(
		mysql.NewConfig("root:root@tcp(192.168.1.66:3306)/zdk_iot?charset=utf8"),
	)
	defer clientStore.Close()
	//映射客户端信息存储接口
	manager.MapClientStorage(clientStore)

	//映射访问令牌生成接口
	manager.MapAccessGenerate(generates.NewJWTAccessGenerate([]byte("00000000"), jwt.SigningMethodHS512))

	//创建Server实例
	srv := server.NewServer(server.NewConfig(), manager)

	//根据请求的用户名和密码获取用户标识
	srv.SetPasswordAuthorizationHandler(models.CheckAuth)

	//获取请求的用户标识
	srv.SetUserAuthorizationHandler(userAuthorizeHandler)

	//内部错误处理
	srv.SetInternalErrorHandler(func(err error) (re *errors.Response) {
		log.Println("Internal Error:", err.Error())
		return
	})

	//响应错误处理(支持自定义URI及错误明细)
	srv.SetResponseErrorHandler(func(re *errors.Response) {
		log.Println("Response Error:", re.Error.Error())
	})

	//srv.SetAllowGetAccessRequest(true)
	//srv.SetClientInfoHandler(server.ClientFormHandler)
	////检查是否允许该客户端通过该授权模式请求令牌
	//srv.SetClientAuthorizedHandler(handler.ClientAuthorizedHandler)
	////检查该客户端所申请的权限范围
	//srv.SetClientScopeHandler(handler.ClientScopeHandler)
	////获取请求的客户端信息(默认支持：ClientFormHandler,ClientBasicHandler)
	//srv.SetClientInfoHandler(handler.ClientInfoHandler)

	http.HandleFunc("/login", loginHandler)
	http.HandleFunc("/auth", authHandler)

	//授权请求处理
	http.HandleFunc("/authorize", func(w http.ResponseWriter, r *http.Request) {
		store, err := session.Start(nil, w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}

		var form url.Values
		if v, ok := store.Get("ReturnUri"); ok {
			form = v.(url.Values)
		}
		r.Form = form

		fmt.Println("return url:")
		fmt.Println(store.Get("ReturnUri"))

		store.Delete("ReturnUri")
		store.Save()

		//授权请求处理
		err = srv.HandleAuthorizeRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
	})

	//令牌请求处理
	http.HandleFunc("/token", func(w http.ResponseWriter, r *http.Request) {
		//令牌请求处理
		err := srv.HandleTokenRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
		}
	})

	http.HandleFunc("/test", func(w http.ResponseWriter, r *http.Request) {
		token, err := srv.ValidationBearerToken(r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
			return
		}

		data := map[string]interface{}{
			"expires_in": int64(token.GetAccessCreateAt().Add(token.GetAccessExpiresIn()).Sub(time.Now()).Seconds()),
			"client_id":  token.GetClientID(),
			"user_id":    token.GetUserID(),
		}
		e := json.NewEncoder(w)
		e.SetIndent("", "  ")
		e.Encode(data)
	})

	log.Println("Server is running at 9096 port.")
	log.Fatal(http.ListenAndServe(":9096", nil))
}

func userAuthorizeHandler(w http.ResponseWriter, r *http.Request) (userID string, err error) {
	store, err := session.Start(nil, w, r)
	if err != nil {
		return
	}

	uid, ok := store.Get("LoggedInUserID")
	if !ok {
		if r.Form == nil {
			r.ParseForm()
		}

		fmt.Println("user auth:")
		fmt.Println(r.Form)

		store.Set("ReturnUri", r.Form)
		store.Save()
		w.Header().Set("Location", "/login")
		w.WriteHeader(http.StatusFound)
		return
	}

	userID = uid.(string)
	store.Delete("LoggedInUserID")
	store.Save()
	return
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
	store, err := session.Start(nil, w, r)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	if r.Method == "POST" {
		username := r.PostFormValue("username")
		password := r.PostFormValue("password")

		userID,err := models.CheckAuth(username,password)
		if err != nil{
			fmt.Fprintln(w,err)
		}else if userID == "0"{
			fmt.Fprintln(w,"密码错误")
		}else{
			fmt.Println("login handler user id:")
			fmt.Println(userID)
			store.Set("LoggedInUserID", userID)
			store.Save()

			w.Header().Set("Location", "/auth")
			w.WriteHeader(http.StatusFound)
		}
		fmt.Println(store.Get("LoggedInUserID"))
		return
	}
	outputHTML(w, r, "static/login.html")
}

func authHandler(w http.ResponseWriter, r *http.Request) {
	store, err := session.Start(nil, w, r)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	if _, ok := store.Get("LoggedInUserID"); !ok {
		w.Header().Set("Location", "/login")
		w.WriteHeader(http.StatusFound)
		return
	}

	outputHTML(w, r, "static/auth.html")
}

func outputHTML(w http.ResponseWriter, req *http.Request, filename string) {
	file, err := os.Open(filename)
	if err != nil {
		http.Error(w, err.Error(), 500)
		return
	}
	defer file.Close()
	fi, _ := file.Stat()
	http.ServeContent(w, req, file.Name(), fi.ModTime(), file)
}

```

##### 参考文档
- [OAuth 2.0 的四种方式](https://www.jianshu.com/p/8f878e5537da)
- [OAuth2介绍与使用](https://www.jianshu.com/p/a75c59a29bdf)
- [如何用GO实现OAuth2授权功能](https://www.jianshu.com/p/641761c3591c)
- [Golang OAuth2 Server Framework](https://go-oauth2.github.io/zh/)
- [Go学习笔记(七) | 理解OAuth 2.0并实现一个客户端](https://razeencheng.com/post/oauth2-protocol-details)
- [oauth2-server](http://gitlab.myschools.me/suguo.yao/oauth-service/tree/master)
- [How to get JSON response in Golang](https://stackoverflow.com/questions/17156371/how-to-get-json-response-in-golang)
- [golang中发送http请求的几种常见情况](https://studygolang.com/articles/4489)
- [用单元测试方法测一个http handler](https://gocn.vip/question/1031)