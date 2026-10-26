### MetalLB

使用 [MetalLB](https://metallb.universe.tf/installation/) 让 `LoadBalancer` 服务使用 `EXTERNAL-IP`

在 `k8s-master` 安装 `MetalLB`

> https://github.com/metallb/metallb
>
> https://metallb.universe.tf/installation/
>
>MetalLB: v0.16.1
>
>k8s版本：k8s-v1.33.1

```shell
kubectl get configmap -n kube-system kube-proxy
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

```shell
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

```shell
kubectl rollout restart daemonset kube-proxy -n kube-system
```

**配置 MetalLB 为Layer2 模式**

```shell
# kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-native.yaml

mkdir -p ~/MetalLB-yml && cd ~/MetalLB-yml

wget https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-native.yaml

# 查看镜像

root@k8s-master01:~/MetalLB-yml# grep image: metallb-native.yaml 
        image: quay.io/metallb/controller:v0.16.1
        image: quay.io/metallb/speaker:v0.16.1

# 替换镜像
sed -i 's#quay.io/metallb#registry.cn-hangzhou.aliyuncs.com/mcalico#g' metallb-native.yaml

# 查看替换镜像
root@k8s-master01:~/MetalLB-yml# grep image: metallb-native.yaml 
        image: registry.cn-hangzhou.aliyuncs.com/mcalico/controller:v0.16.1
        image: registry.cn-hangzhou.aliyuncs.com/mcalico/speaker:v0.16.1

# 部署
kubectl apply -f ~/MetalLB-yml/metallb-native.yaml


# 查看是否正常运行

root@k8s-master01:~/MetalLB-yml# kubectl get pod -n metallb-system 
NAME                          READY   STATUS    RESTARTS   AGE
controller-557859d9cc-5z9vx   1/1     Running   0          41s
speaker-bfkv4                 1/1     Running   0          41s
speaker-bh4bp                 1/1     Running   0          41s
speaker-h6kzj                 1/1     Running   0          41s
speaker-hwmcz                 1/1     Running   0          41s
speaker-zn6m7                 1/1     Running   0          41s
```

```shell
# k8s-master 创建ip地址池
# 192.168.3.180-189 用于metallb VIP
cat > ~/MetalLB-yml/iptool.yaml << 'EOF'
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 192.168.3.180-192.168.3.189 # 网段跟node节点保持一致
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default
EOF

kubectl apply -f ~/MetalLB-yml/iptool.yaml
```

```shell
root@k8s-master01:~/MetalLB-yml# kubectl get IPAddressPool -n metallb-system
NAME      AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
default   true          false             ["192.168.3.180-192.168.3.189"]

```

