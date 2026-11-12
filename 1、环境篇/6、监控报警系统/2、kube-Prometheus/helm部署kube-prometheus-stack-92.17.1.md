
helm部署kube-prometheus-stack-92.17.1

kube-prometheus-stack chart包地址：https://github.com/prometheus-community/helm-charts/releases
kube-prometheus 地址：https://github.com/prometheus-operator/kube-prometheus/


# 下载
国内很难下载到
```shell
mkdir -p ~/kube-prometheus-stack && cd ~/kube-prometheus-stack
wget https://gh-proxy.org/https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-92.17.1/kube-prometheus-stack-92.17.1.tgz
```

# 解压安装

```shell

tar xf kube-prometheus-stack-92.17.1.tgz
cd kube-prometheus-stack/

```


查看镜像
```shell
#  查看镜像
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# helm template kube-prometheus-stack . \
  --namespace monitoring \
  -f values.yaml | \
  grep -oP '(quay|docker|ghcr|k8s\.gcr|registry\.k8s)\.io/[^\s"'\''>]+' | \
  sort -u > images1.txt
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# cat images1.txt 
docker.io/bats/bats:1.14.0
docker.io/grafana/grafana:13.3.0-distroless
ghcr.io/jkroepke/kube-webhook-certgen:1.11.0
quay.io/kiwigrid/k8s-sidecar:2.14.3
quay.io/prometheus/alertmanager:v0.34.2
quay.io/prometheus/node-exporter:v1.12.1-distroless
quay.io/prometheus-operator/prometheus-config-reloader:v0.94.2
quay.io/prometheus-operator/prometheus-operator:v0.94.2
quay.io/prometheus/prometheus:v3.15.1-distroless
quay.io/thanos/thanos:v0.42.5
registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.20.0

#
 m命令解释
-o：只输出匹配的部分。
-P：使用 Perl 正则表达式。
[^\s"'\''>]+：匹配非空白、非引号、非 > 的字符，确保提取完整的镜像地址。
这种方法通常比 sed 更健壮，因为它直接提取匹配的字符串，而不需要处理复杂的替换逻辑。
`
``

| 组成部分 | 含义 | 详细解释 |
| --- | --- | --- |
| [ ... ] | ‌字符类‌ | 表示匹配方括号内定义的‌任意一个‌字符集合。 |
| ^ | ‌取反符‌ | 位于方括号内的开头，表示‌否定‌。即匹配‌不在‌后续列表中的字符。 |
| \s | ‌空白字符‌ | 匹配任何空白字符，包括空格、制表符 (\t)、换行符 (\n)等。 |
| " | ‌双引号‌ | 匹配双引号字符。YAML 或 JSON 中字符串常由双引号包裹。 |
| ' | ‌单引号‌ | 匹配单引号字符。Shell 脚本或某些配置中字符串常由单引号包裹。(注：在 Shell 命令中写作 '\'' 是为了转义单引号，实际正则匹配的是 ') |
| > | ‌大于号‌ | 匹配 > 字符。在 HTML/XML 标签或某些日志格式中，> 常作为字段结束符。 |
| + | ‌量词‌ | 表示前面的字符类 [...] 必须出现‌一次或多次‌。这确保了提取的是一个完整的连续字符串，而不是单个字符。 |

```shell
#   镜像都是国外的 
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# cat images1.txt 
docker.io/bats/bats:1.14.0
docker.io/grafana/grafana:13.3.0-distroless
docker.io/library/busybox:1.38.0
ghcr.io/jkroepke/kube-webhook-certgen:1.11.0
quay.io/kiwigrid/k8s-sidecar:2.14.3
quay.io/prometheus/alertmanager:v0.34.2
quay.io/prometheus/node-exporter:v1.12.1-distroless
quay.io/prometheus-operator/prometheus-config-reloader:v0.94.2
quay.io/prometheus-operator/prometheus-operator:v0.94.2
quay.io/prometheus/prometheus:v3.15.1-distroless
quay.io/thanos/thanos:v0.42.5
registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.20.0

# 替换成国内镜像

  docker.io/bats/bats:1.14.0: registry.cn-hangzhou.aliyuncs.com/mcalico/bats:1.14.0
  docker.io/grafana/grafana:13.3.0-distroless: registry.cn-hangzhou.aliyuncs.com/mcalico/grafana:13.3.0-distroless
  docker.io/library/busybox:1.38.0: registry.cn-hangzhou.aliyuncs.com/mcalico/busybox:1.38.0
  ghcr.io/jkroepke/kube-webhook-certgen:1.11.0: registry.cn-hangzhou.aliyuncs.com/mcalico/kube-webhook-certgen:1.11.0
  quay.io/kiwigrid/k8s-sidecar:2.14.3: registry.cn-hangzhou.aliyuncs.com/mcalico/k8s-sidecar:2.14.3
  quay.io/prometheus/alertmanager:v0.34.2: registry.cn-hangzhou.aliyuncs.com/mcalico/alertmanager:v0.34.2
  quay.io/prometheus/node-exporter:v1.12.1-distroless: registry.cn-hangzhou.aliyuncs.com/mcalico/node-exporter:v1.12.1-distroless
  quay.io/prometheus-operator/prometheus-config-reloader:v0.94.2: registry.cn-hangzhou.aliyuncs.com/mcalico/prometheus-config-reloader:v0.94.2 
  quay.io/prometheus-operator/prometheus-operator:v0.94.2: registry.cn-hangzhou.aliyuncs.com/mcalico/prometheus-operator:v0.94.2
  quay.io/prometheus/prometheus:v3.15.1-distroless: registry.cn-hangzhou.aliyuncs.com/mcalico/prometheus:v3.15.1-distroless
  registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.20.0: registry.cn-hangzhou.aliyuncs.com/mcalico/kube-state-metrics:v2.20.0
  registry.k8s.io/kubectl:v1.35.9: registry.cn-hangzhou.aliyuncs.com/mcalico/kubectl:v1.35.9

```


```shell
cat >prod-values.yaml <<EOF
# ============================================================
# 镜像替换：全部指向阿里云杭州仓库 mcalico
# 注意：global.imageRegistry 不可靠，以下均为逐组件覆盖
# ============================================================

crds:
  enabled: true
  upgradeJob:
    enabled: false

    forceConflicts: true
    image:
      busybox:
        registry: registry.cn-hangzhou.aliyuncs.com
        repository: mcalico/busybox
        tag: "1.38.0"
        sha: ""
        pullPolicy: IfNotPresent
      kubectl:
        registry: registry.cn-hangzhou.aliyuncs.com
        repository: mcalico/kubectl
        tag: "v1.35.9"
        pullPolicy: IfNotPresent
# ---------- Prometheus Operator ----------
prometheusOperator:
  image:
    registry: registry.cn-hangzhou.aliyuncs.com
    repository: mcalico/prometheus-operator
    tag: v0.94.2

  # Config Reloader（与 Operator 同版本，Chart 自动使用 Operator Tag）
  prometheusConfigReloader:
    image:
      registry: registry.cn-hangzhou.aliyuncs.com
      repository: mcalico/prometheus-config-reloader
      # tag 无需显式设置，默认跟随 Operator 的 tag

  # Admission Webhook 证书生成 Job（原 ghcr.io/jkroepke/kube-webhook-certgen）
  admissionWebhooks:
    patch:
      image:
        registry: registry.cn-hangzhou.aliyuncs.com
        repository: mcalico/kube-webhook-certgen
        tag: "1.11.0"

# ---------- Prometheus ----------
prometheus:
  prometheusSpec:
    image:
      registry: registry.cn-hangzhou.aliyuncs.com
      repository: mcalico/prometheus
      tag: v3.15.1-distroless
    # 核心：启用持久化存储
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi # 根据实际数据量调整
          storageClassName: "nfs-storage"
    # 保留策略：时间与容量双限制，防止磁盘写满
    retention: 15d           # 保留15天数据
    retentionSize: 45GiB     # 数据量达到45GiB时开始清理旧数据，需小于PVC容量  

  # 确保 externalUrl 与外部访问地址一致，避免告警等链接错误
  externalUrl: "http://prometheus.jiamiao.com"
  route:
    main:
      enabled: true
      apiVersion: gateway.networking.k8s.io/v1
      kind: HTTPRoute
      hostnames:
        - "prometheus.jiamiao.com"
      parentRefs:
        - name: jiamiaogateway   # 你的 Gateway 名称
          namespace: default      # Gateway 所在命名空间
          sectionName: default    # 你的 Listener 名称
      matches:
        - path:
            type: PathPrefix
            value: /    

# ---------- Alertmanager ----------
alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi
          storageClassName: "nfs-storage"  
    image:
      registry: registry.cn-hangzhou.aliyuncs.com
      repository: mcalico/alertmanager
      tag: v0.34.2
  externalUrl: "http://alertmanager.jiamiao.com"
  route:
    main:
      enabled: true
      apiVersion: gateway.networking.k8s.io/v1
      kind: HTTPRoute
      hostnames:
        - "alertmanager.jiamiao.com"
      parentRefs:
        - name: jiamiaogateway
          namespace: default
          sectionName: default
      matches:
        - path:
            type: PathPrefix
            value: /      

# ---------- Node Exporter ----------
prometheus-node-exporter:
  image:
    registry: registry.cn-hangzhou.aliyuncs.com
    repository: mcalico/node-exporter
    tag: v1.12.1

# ---------- Kube State Metrics ----------
kube-state-metrics:
  image:
    registry: registry.cn-hangzhou.aliyuncs.com
    repository: mcalico/kube-state-metrics
    tag: v2.20.0

# ---------- Grafana ----------
grafana:
defaultDashboardsTimezone: "Asia/Shanghai"
    persistence:
    enabled: true
    size: 10Gi
    storageClassName: "nfs-storage"
  image:
    registry: registry.cn-hangzhou.aliyuncs.com
    repository: mcalico/grafana
    tag: 13.3.0-distroless
  testFramework:
    image:
      registry: registry.cn-hangzhou.aliyuncs.com
      repository: mcalico/bats
      tag: "1.14.0"

  initChownData:
    image:
      registry: registry.cn-hangzhou.aliyuncs.com
      repository: mcalico/busybox
      tag: "1.38.0"
      pullPolicy: IfNotPresent

  # Grafana 的 Sidecar（原 quay.io/kiwigrid/k8s-sidecar）
  sidecar:
    image:
      registry: registry.cn-hangzhou.aliyuncs.com
      repository: mcalico/k8s-sidecar
      tag: "2.14.3"
  # Grafana 的 externalUrl 通常由其自身配置管理，可按需设置
  grafana.ini:
    server:
      domain: "grafana.jiamiao.com"
      root_url: "http://grafana.jiamiao.com/" 
  route:
    date_formats:
      # 将默认时区设置为上海，也可以使用 "browser" 让浏览器决定
      default_timezone: "Asia/Shanghai"      
    main:
      enabled: true
      apiVersion: gateway.networking.k8s.io/v1
      kind: HTTPRoute
      hostnames:
        - "grafana.jiamiao.com"
      parentRefs:
        - name: jiamiaogateway
          namespace: default
          sectionName: default
      matches:
        - path:
            type: PathPrefix
            value: /

EOF
```


```shell

# 安装到 monitoring 命名空间
helm upgrade  --install kube-prometheus-stack ./ \
  --namespace monitoring \
  --create-namespace \
  -f prod-values.yaml

# 安装成功显示
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# helm upgrade  --install kube-prometheus-stack ./ \
  --namespace monitoring \
  --create-namespace \
  -f prod-values.yaml
Release "kube-prometheus-stack" does not exist. Installing it now.
NAME: kube-prometheus-stack
LAST DEPLOYED: Fri Nov  6 17:11:06 2026
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=kube-prometheus-stack"

Get Grafana 'admin' user password by running:

  kubectl --namespace monitoring get secrets kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kube-prometheus-stack" -oname)
  kubectl --namespace monitoring port-forward $POD_NAME 3000

Get your grafana admin user password by running:

  kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo


Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

```


查看IP和域名

```shell
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# kubectl get gateway -A
NAMESPACE   NAME             CLASS   ADDRESS         PROGRAMMED   AGE
default     jiamiaogateway   nginx   192.168.3.181   True         13d
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# kubectl get httproute -A
NAMESPACE    NAME                                 HOSTNAMES                      AGE
monitoring   kube-prometheus-stack-alertmanager   ["alertmanager.jiamiao.com"]   14s
monitoring   kube-prometheus-stack-grafana        ["grafana.jiamiao.com"]        14s
monitoring   kube-prometheus-stack-prometheus     ["prometheus.jiamiao.com"]     14s

```
配置本地解析

```shell

修改hosts文件```

192.168.3.181 prometheus.jiamiao.com alertmanager.jiamiao.com grafana.jiamiao.com
```shell

访问

prometheus:   http://prometheus.jiamiao.com
alertmanager:  http://alertmanager.jiamiao.com
grafana:           http://grafana.jiamiao.com

```shell
查看grafana密码
root@k8s-master01:~/kube-prometheus-stack/kube-prometheus-stack# kubectl --namespace monitoring get secrets kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
5oLovR1oYSs5Q5Fd2hJhUTYfb1D9JUmgisZn9anh


```



# 卸载
helm uninstall kube-prometheus-stack    --namespace monitoring

```
