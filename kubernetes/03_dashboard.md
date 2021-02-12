
ref : [[Day 14] Kubernetes Dashboard 介紹](https://ithelp.ithome.com.tw/articles/10195385)

# minikube 安裝 dashboard

```
$ minikube dashboard --url
http://172.17.0.2:xxxxx
```

但使用 minikube 的方式`只能在本機瀏覽`

```
ubuntu@ip-10-0-50-61:~$ netstat -tunl

Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 127.0.0.1:32774         0.0.0.0:*               LISTEN
```


# kubectl 安裝 Dashboard

一般 Kubernetes `預設不會有` Dashboard 套件  
需要透過以下的指令，透過 kubectl apply 指令創建 dashboard service  

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml
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
http://k8s-dashboard.scottchayaa.com/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/


# Bearer Token

## Create Service Account
kubectl apply -f dashboard-adminuser.yaml
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

Now we need to find token we can use to log in. Execute following command:

```sh
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

```
Name:         admin-user-token-drl6h
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 8ecc395c-8b90-4ef0-bb3e-aae871432b9a

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Ikl6ZTVWeHJCQnFxSUZmT2lZWE5xNEpiNWNzQl9obThCOG5Idi0zZU9UdWMifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWRybDZoIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4ZWNjMzk1Yy04YjkwLTRlZjAtYmIzZS1hYWU4NzE0MzJiOWEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.u9l3dPpZ81iomTjxJWbWQav2qS3GB8ZNPHMVbmqXDb4NjZCaFMOF0VXl4BqnukGQp_8PNWjseq53pimTPRIUSndxz-DJfFn2FenUs5oNn-bzXARFGvbrAPjJ4ArteW78pogjSNQzLHCz6BjEuuU_St_diT-SW4WSo7BX7mc3s6klgYydzE1dwHTxVfBxefef1lvv0XCqEsMGUy_K4xsmm7ax2Kz4GXKQGWJJHlNyvQLk_dzY8q6uCjt6smtqjZanyRz7sfDpw_znEWTVUSN3e1-Tm6gmR-H6gpbILy4i7vi3bdE3FdHNH0E44ICc536xhbUuNL9r2jE-N4NXcpvXnA
```