
metrics-server-v0.9.1

https://github.com/kubernetes-sigs/metrics-server

### Compatibility Matrix

Metrics Server | Metrics API group/version | Supported Kubernetes version
---------------|---------------------------|-----------------------------
0.9.x          | `metrics.k8s.io/v1beta1`  | 1.34+
0.8.x          | `metrics.k8s.io/v1beta1`  | 1.31+
0.7.x          | `metrics.k8s.io/v1beta1`  | 1.27+
0.6.x          | `metrics.k8s.io/v1beta1`  | 1.25+
0.5.x          | `metrics.k8s.io/v1beta1`  | *1.8+
0.4.x          | `metrics.k8s.io/v1beta1`  | *1.8+
0.3.x          | `metrics.k8s.io/v1beta1`  | 1.8-1.21


## 安装metrics-server
### 下载yaml

```shell
mkdir -p ~/metrics-server && cd ~/metrics-server
wget https://gh-proxy.org/https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.9.1/components.yaml

```

### 修改配置

```shell

#1、添加"- --kubelet-insecure-tls"参数（匹配行后）
sed -i '/15s/a\        - --kubelet-insecure-tls' ~/metrics-server/components.yaml

#2、 修改镜像
sed -i 's#registry.k8s.io/metrics-server/metrics-server:v0.9.1#registry.cn-hangzhou.aliyuncs.com/mcalico/metrics-server:v0.9.1#g' components.yaml

```

### 部署

```shell
kubectl apply -f ~/metrics-server/components.yaml

kubectl get pods -n kube-system -l k8s-app=metrics-server
```

### 查询是否正常运行

```shell
root@k8s-master01:~/metrics-server# kubectl get pods -n kube-system -l k8s-app=metrics-server
NAME                              READY   STATUS    RESTARTS   AGE
metrics-server-85885cc7fd-rw22g   1/1     Running   0          106s

root@k8s-master01:~/metrics-server# kubectl top nodes
NAME           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
k8s-master01   126m         6%       1213Mi          85%         
k8s-master02   105m         5%       991Mi           69%         
k8s-master03   102m         5%       1108Mi          78%         
k8s-worker01   58m          2%       1185Mi          83%         
k8s-worker02   54m          2%       1251Mi          88%         
k8s-worker03   73m          0%       1520Mi          9%          
root@k8s-master01:~/metrics-server# kubectl top pod
NAME                                        CPU(cores)   MEMORY(bytes)   
gateway-deployment-5c98f86997-9jm8w         0m           19Mi            
gateway-deployment-5c98f86997-nt5gv         0m           15Mi            
gateway-deployment-5c98f86997-szhq2         0m           6Mi             
jiamiaogateway-nginx-69487f6f95-7b9lm       8m           84Mi            
jiamiaogateway-nginx-69487f6f95-j8v9p       8m           84Mi            
jiamiaogateway-nginx-69487f6f95-sqxtn       10m          89Mi            
nginx-ingress-deployment-559b54fc7c-lqmsm   0m           7Mi             
nginx-ingress-deployment-559b54fc7c-n4ld7   0m           12Mi            
nginx-ingress-deployment-559b54fc7c-xjcpb   0m           3Mi    ```

