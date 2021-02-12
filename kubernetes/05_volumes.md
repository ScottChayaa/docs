# emptyDir



# hostPath
hostPath 的生命週期與 Node 相同  
當 Pod 因某些原因而須重啟時，檔案仍會保存在 Node 的檔案系統(file system)底下  
直到該 Node 物件被 Kubernetes Cluster 移除，資料才會消失  

hostpath-example.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apiserver
spec:
  containers:
  - name: apiserver
    image: zxcvbnius/docker-demo
    volumeMounts:
    - mountPath: /tmp
      name: tmp-volume
    imagePullPolicy: Always
  volumes:
  - name: tmp-volume
    hostPath:
      path: /tmp
      type: Directory
```