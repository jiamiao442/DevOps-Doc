nfs-storage

> k8s（pv 与 pvc）动态存储 StorageClass（nfs动态存储）
>
> k8s版本：k8s-v1.35.9
>
> 版本：nfs-subdir-external-provisioner-4.0.18
>
> https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

1、部署nfs

nfs 服务端（k8s-master）

```shell
# 所有服务端节点安装nfs
sudo apt update
sudo apt install -y nfs-kernel-server

systemctl enable nfs-server rpcbind --now

root@k8s-master01:~# systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2026-11-03 16:28:10 CST; 19s ago
   Main PID: 228881 (code=exited, status=0/SUCCESS)
        CPU: 9ms

Nov 03 16:28:10 k8s-master01 systemd[1]: Starting nfs-server.service - NFS server and services...
Nov 03 16:28:10 k8s-master01 exportfs[228880]: exportfs: can't open /etc/exports for reading
Nov 03 16:28:10 k8s-master01 systemd[1]: Finished nfs-server.service - NFS server and services.


# 创建nfs共享目录、授权

sudo mkdir -p /nfs-storage/k8s-pvs
sudo chown nobody:nogroup /nfs-storage/k8s-pvs
sudo chmod 777 /nfs-storage/k8s-pvs


# 写入exports
cat > /etc/exports << EOF
/nfs-storage/k8s-pvs 192.168.3.0/24(rw,sync,no_root_squash,no_subtree_check)
EOF
 
systemctl reload nfs-server
 
使用如下命令进行验证
root@k8s-master01:~#  showmount -e 192.168.3.61
Export list for 192.168.3.61:
/nfs-storage/k8s-pvs 192.168.3.0/24
```

nfs 客户端（k8s-node）

```shell
sudo apt update
sudo apt install -y nfs-common
 
systemctl enable rpcbind --now
 
使用如下命令进行验证
# showmount -e 192.168.3.61
Export list for 192.168.3.61:
/nfs-storage/k8s-pvs 192.168.3.0/24

```

备份

```shell
mkdir -p /nfs-storage/k8s-pvs && chmod -R 777 /nfs-storage/k8s-pvs

rsync -avzP /nfs-storage/k8s-pvs root@192.168.3.62:/nfs-storage

00 2 * * * rsync -avz /nfs-storage/k8s-pvs root@192.168.3.62:/nfs-storage &>/dev/null
```

2、动态创建 NFS存储（动态存储）

> https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

```shell
mkdir ~/nfs-subdir-external-provisioner-4.0.18 && cd ~/nfs-subdir-external-provisioner-4.0.18
```

> 版本：nfs-subdir-external-provisioner-4.0.18
>
> https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/tree/nfs-subdir-external-provisioner-4.0.18/deploy

```shell
# wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/raw/nfs-subdir-external-provisioner-4.0.18/deploy/deployment.yaml
 
# wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/raw/nfs-subdir-external-provisioner-4.0.18/deploy/rbac.yaml
 
# wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/raw/nfs-subdir-external-provisioner-4.0.18/deploy/class.yaml
```

```shell
wget https://gitee.com/kubelsp/upload/raw/master/nfs-storage/v4.0.18/deployment.yaml
 
wget https://gitee.com/kubelsp/upload/raw/master/nfs-storage/v4.0.18/rbac.yaml
 
wget https://gitee.com/kubelsp/upload/raw/master/nfs-storage/v4.0.18/class.yaml
```

```shell
# 1、修改镜像（默认谷歌k8s.gcr.io）
sed -i 's#registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2#ccr.ccs.tencentyun.com/huanghuanhui/nfs-storage:v4.0.2#g' deployment.yaml
# 2、修改nfs服务端地址
sed -i 's/10.3.243.101/192.168.3.61/g' deployment.yaml
# 3、修改存储地址（/nfs-storage/k8s-pvs）
sed -i 's#\/ifs\/kubernetes#\/nfs-storage\/k8s-pvs#g' deployment.yaml

sed -i 's#nfs-client#nfs-storage#g' class.yaml

sed -i 's/namespace: default/namespace: nfs-storage/g' rbac.yaml deployment.yaml
```

> 使用这个镜像：ccr.ccs.tencentyun.com/huanghuanhui/nfs-storage:v4.0.2 

```shell
kubectl create ns nfs-storage

kubectl -n nfs-storage apply -f .

kubectl get pods -n nfs-storage -l app=nfs-client-provisioner

kubectl get storageclass
```

```shell
# 将 StorageClass 标记为默认

kubectl patch storageclass nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
