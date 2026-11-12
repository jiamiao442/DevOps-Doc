

# ingress-nginx-v1.15.1

参考地址： https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md#quick-start

```shell
k8s： v1.35.9
ingres-nginx : v1.15.1   (这是ingress-nginx的最后一个版本，github已经归档了，提示： This repo was archived by the owner on Mar 24, 2026. It is now read-only.)
helm ：v4.3.1
chart version： ingress-nginx/ingress-nginx --version 4.15.1
```

```powershell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update

helm search repo ingress-nginx/ingress-nginx

helm pull ingress-nginx/ingress-nginx --version 4.15.1 --untar
```

```shell
root@k8s-master01:~/ingress-nginx# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories

root@k8s-master01:~# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. ⎈Happy Helming!⎈

root@k8s-master01:~# helm search repo ingress-nginx/ingress-nginx
NAME                       	CHART VERSION	APP VERSION	DESCRIPTION                                       
ingress-nginx/ingress-nginx	4.15.1       	1.15.1     	Ingress controller for Kubernetes using NGINX a...

root@k8s-master01:~# helm pull ingress-nginx/ingress-nginx --version 4.15.1 --untar

root@k8s-master01:~# ls ingress-nginx/
changelog  Chart.yaml  ci  cloudbuild.yaml  OWNERS  README.md  README.md.gotmpl  templates  tests  values.yaml


```

国内镜像
```shelll
#  ingress-nginx v1.5.1 images  last version
  registry.k8s.io/ingress-nginx/controller:v1.15.1
  registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.6.9
  registry.k8s.io/defaultbackend-amd64:1.5

# 国内阿里云镜像  
  registry.cn-hangzhou.aliyuncs.com/mcalico/ingress-nginx-controller:v1.15.1
  registry.cn-hangzhou.aliyuncs.com/mcalico/kube-webhook-certgen:v1.6.9
  registry.cn-hangzhou.aliyuncs.com/mcalico/defaultbackend-amd64:1.5
```
## `方式2：MetalLB + Deployment + LoadBalancer（多副本、高可用）`
```shell 
cat >> ~/ingress-nginx/values-prod.yaml << 'EOF'
global:
  image:
    registry: registry.cn-hangzhou.aliyuncs.com

controller:
  name: controller
  image:

    image: mcalico/ingress-nginx-controller
    tag: "v1.15.1"
    digest:
    digestChroot:
    pullPolicy: IfNotPresent

  metrics:
    enabled: true
  #  serviceMonitor:
  #    enabled: true 

  kind: Deployment
  replicaCount: 3  # 设置副本数为 3

  affinity: # 设置软策略
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
              - ingress-nginx
          topologyKey: kubernetes.io/hostname
        weight: 100

  admissionWebhooks: # 强烈建议开启 admission webhook
    enabled: true
    patch:
      enabled: true
      image:
        image: mcalico/kube-webhook-certgen
        tag: v1.6.9
        digest:
        pullPolicy: IfNotPresent

defaultBackend:
  enabled: true
  name: defaultbackend
  image:
    image: mcalico/defaultbackend-amd64
    tag: "1.5"
    digest:
    pullPolicy: IfNotPresent
EOF
```

```shell
kubectl create ns ingress-nginx

helm upgrade --install  ingress-nginx .  -f ./values-prod.yaml  --namespace ingress-nginx
```

```shell

root@k8s-master01:~/ingress-nginx# helm upgrade --install  ingress-nginx .  -f ./values-prod.yaml  --namespace ingress-nginx
Release "ingress-nginx" does not exist. Installing it now.
NAME: ingress-nginx
LAST DEPLOYED: Wed Oct 28 14:41:24 2026
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the load balancer IP to be available.
You can watch the status by running 'kubectl get service --namespace ingress-nginx ingress-nginx-controller --output wide --watch'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
root@k8s-master01:~/ingress-nginx# 

#  查看pod运行
root@k8s-master01:~/ingress-nginx# kubectl get pod --namespace ingress-nginx 
NAME                                            READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-86b6d577c9-kx5nb       1/1     Running   0          2m41s
ingress-nginx-controller-86b6d577c9-r4w56       1/1     Running   0          2m41s
ingress-nginx-controller-86b6d577c9-s7qb8       1/1     Running   0          2m41s
ingress-nginx-defaultbackend-6bd8b89cf5-wq74f   1/1     Running   0          2m41s
root@k8s-master01:~/ingress-nginx# kubectl get pod --namespace ingress-nginx  -o wide
NAME                                            READY   STATUS    RESTARTS   AGE    IP              NODE           NOMINATED NODE   READINESS GATES
ingress-nginx-controller-86b6d577c9-kx5nb       1/1     Running   0          3m5s   10.244.79.93    k8s-worker01   <none>           <none>
ingress-nginx-controller-86b6d577c9-r4w56       1/1     Running   0          3m5s   10.244.69.221   k8s-worker02   <none>           <none>
ingress-nginx-controller-86b6d577c9-s7qb8       1/1     Running   0          3m5s   10.244.39.226   k8s-worker03   <none>           <none>
ingress-nginx-defaultbackend-6bd8b89cf5-wq74f   1/1     Running   0          3m5s   10.244.39.225   k8s-worker03   <none>           <none>

# 查看service
root@k8s-master01:~/ingress-nginx# kubectl get service --namespace ingress-nginx 
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.109.70.192    192.168.3.180   80:30239/TCP,443:32560/TCP   4m10s
ingress-nginx-controller-admission   ClusterIP      10.102.157.187   <none>          443/TCP                      4m10s
ingress-nginx-controller-metrics     ClusterIP      10.97.119.61     <none>          10254/TCP                    4m10s
ingress-nginx-defaultbackend         ClusterIP      10.109.76.167    <none>          80/TCP                       4m10s
root@k8s-master01:~/ingress-nginx# kubectl get service --namespace ingress-nginx ingress-nginx-controller
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.109.70.192   192.168.3.180   80:30239/TCP,443:32560/TCP   6m3s

# 测试访问
root@k8s-master01:~/ingress-nginx# curl http://192.168.3.180
default backend - 404

# 查看ingressClass
root@k8s-master01:~# kubectl get ingressclass
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       132m

```

卸载

```shell
[root@k8s-master ~/ingress-nginx]# helm delete ingress-nginx -n ingress-nginx

[root@k8s-master ~/ingress-nginx]# kubectl delete ns ingress-nginx
```

### Ingress基础案例配置

本地新建YAML清单，定义Deployment资源描述模板。
```shell
root@k8s-master01:~# vim nginx-ingress-deployment.yml

# 写入下列内容
apiVersion: apps/v1		# 固定配置
kind: Deployment		# 资源类型
metadata:
  name: nginx-ingress-deployment
  labels:					# 定义标签
    app: nginx-ingress-deployment	# 自定义标签（键和值均可自定义）
spec:
  replicas: 3				# 预期数量为3
  selector:
    matchLabels:			# 标签匹配模式：精确匹配
      app: nginx-ingress-deployment	# 匹配的标签值
  template:			# 根据此模板来新建Pod
    metadata:
      labels:		# 给Pod加上标签，需要和selector定义一致
        app: nginx-ingress-deployment
    spec:
      containers:
        - name: nginx
          image: registry.cn-hangzhou.aliyuncs.com/mcalico/nginx:1.31.8
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80

```

基于YAML清单向Kubernetes集群声明式地创建Deployment资源。

```shell
kubectl create -f nginx-ingress-deployment.yml
```

```shell
root@k8s-master01:~# kubectl create -f nginx-ingress-deployment.yml
deployment.apps/nginx-ingress-deployment created

```

验证：查看当前集群中的所有Deployment资源。

```shell
root@k8s-master01:~# kubectl get deployments.apps 
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
nginx-ingress-deployment   3/3     3            3           2m43s

```

本地新建YAML清单，定义ClusterIP类型的Service资源描述模板。
supported values: "ClusterIP", "ExternalName", "LoadBalancer", "NodePort"

```shell
root@k8s-master01:~# vim nginx-ingress-service.yml

# 写入下列内容
apiVersion: v1		# 固定配置
kind: Service		# 资源类型
metadata:
  name: nginx-ingress-service
spec:
  # Service类型，裸机需要借助MetalLB来分配IP
  # 如果没有MetalLB，后续Ingress的ADDRESS字段会为空
  type: ClusterIP
  # 标签选择器，匹配后端Pod择器，匹配后端Pod	
  selector:
    # 匹配后端Pod的标签名称对应
    app: nginx-ingress-deployment	
  ports:
    - protocol: TCP		# 使用TCP协议
      port: 80		# Service自己暴露的端口
      targetPort: 80	# 后端Pod实际监听的端口
```

验证service
```shell
root@k8s-master01:~# vim nginx-ingress-service.yml 
root@k8s-master01:~# kubectl apply -f nginx-ingress-service.yml 
service/nginx-ingress-service created
root@k8s-master01:~# kubectl get service
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes              ClusterIP   10.96.0.1        <none>        443/TCP   6d
nginx-ingress-service   ClusterIP   10.102.170.173   <none>        80/TCP    6m25s

```

验证：查看Service关联的后端Pod IP与端口信息。
```shell
kubectl get endpoints
root@k8s-master01:~# kubectl get endpointslice
NAME                          ADDRESSTYPE   PORTS   ENDPOINTS                                  AGE
kubernetes                    IPv4          6443    192.168.3.61,192.168.3.62,192.168.3.63     6d
nginx-ingress-service-tndc8   IPv4          80      10.244.69.223,10.244.79.96,10.244.39.231   2m39s
```

本地新建YAML清单，定义Ingress资源描述模板。

```shell
root@k8s-master01:~# vim nginx-ingress.yml

# 写入下列内容
apiVersion: networking.k8s.io/v1	# 固定配置
kind: Ingress		# 资源类型
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx	# 指定使用nginx-ingress-controller
  rules:			# 路由规则
    - host: www.jiamiao.com	# 当访问www.jiamiao.com时，转发到后端服务 
      http:			
        paths:
          - pathType: Prefix	# 路径匹配规则：前缀匹配
            path: "/"			# 匹配以/开头的路径，也就是全部路径
            backend:			# 匹配成功后，流量转发到哪
              service:			# 转发到Service
                name: nginx-ingress-service		# Service的名称
                port:			# 端口配置
                  number: 80	# 转发Service的80端口

```

```shell
root@k8s-master01:~# vim nginx-ingress.yml
root@k8s-master01:~# kubectl apply -f nginx-ingress.yml 
ingress.networking.k8s.io/nginx-ingress created


```
验证：查看创建的所有的Ingress。

```shell
kubectl get ingress
```
```shell
root@k8s-master01:~# kubectl get ingress
NAME            CLASS   HOSTS             ADDRESS         PORTS   AGE
nginx-ingress   nginx   www.jiamiao.com   192.168.3.180   80      17s
```

验证：本地临时添加域名解析，将域名指向ADDRESS。然后通过域名访问，能够正常显示Nginx测试页。

```shell
echo "192.168.3.180 www.jiamiao.com" >> /etc/hosts
curl http://www.jiamiao.com
```

```shell
root@k8s-master01:~# echo "192.168.3.180 www.jiamiao.com" >> /etc/hosts
root@k8s-master01:
```~# curl http://www.jiamiao.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
