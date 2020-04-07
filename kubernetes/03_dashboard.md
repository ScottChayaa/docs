
ref : [[Day 14] Kubernetes Dashboard 介紹](https://ithelp.ithome.com.tw/articles/10195385)

# minikube 安裝 dashboard

```
$ minikube dashboard --url
http://172.17.0.2:xxxxx
```

但使用 minikube 的方式只能在本機瀏覽

```
ubuntu@ip-10-0-50-61:~$ netstat -tunl

Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 127.0.0.1:32774         0.0.0.0:*               LISTEN
```


# kubectl 安裝 Dashboard

一般 Kubernetes `預設不會有` Dashboard 套件  
需要透過以下的指令，透過 kubectl apply 指令創建 dashboard service  

```sh
kubectl apply -f https://raw.githubusercontent.com/\
```

## 透過 kubectl proxy 連接

安裝完 Dashboard 之後
再透過 kubectl proxy 指令連接到 Dashboard
預設會將 Dashboard 與本機端的 port number 8001 互相 mapping
而開發者可以在本機端上直接透過該 url 存取 Dashboard

```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

但若要將 dashboard 對外開放  
則需再指令後面加上 --address 與 --accept-hosts  

```
kubectl proxy --address='0.0.0.0' --port=8002 --accept-hosts='^*$'
```

```
ubuntu@ip-10-0-50-61:~$ netstat -tunl

Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp6       0      0 :::8002                 :::*                    LISTEN
```


接下來就可以透過設定 nginx 反向代理的方式  
指向內部 k8s dashboard 的 ip:port  

/etc/nginx/conf.d/default.conf
```conf
server {
    listen 80;

    server_name k8s-dashboard.scottchayaa.com;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8002;
    }

}
```

Kubernetes Dashboard url will be like :
http://k8s-dashboard.scottchayaa.com/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

