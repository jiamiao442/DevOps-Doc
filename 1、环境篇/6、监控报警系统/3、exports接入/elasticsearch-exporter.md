###### elasticsearch-exporter

> https://prometheus.io/docs/instrumenting/exporters/
>
> https://github.com/prometheus-community/elasticsearch_exporter

````shell
cat > elasticsearch-exporter.yml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elastic-exporter
  namespace: monitoring
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  selector:
    matchLabels:
      app: elastic-exporter
  template:
    metadata:
      labels:
        app: elastic-exporter
    spec:
      containers:
        - command:
            - /bin/elasticsearch_exporter
            - --es.uri=http://elasticsearch:9200
            - --es.all
          image: quay.io/prometheuscommunity/elasticsearch-exporter:latest
          securityContext:
            capabilities:
              drop:
                - SETPCAP
                - MKNOD
                - AUDIT_WRITE
                - CHOWN
                - NET_RAW
                - DAC_OVERRIDE
                - FOWNER
                - FSETID
                - KILL
                - SETGID
                - SETUID
                - NET_BIND_SERVICE
                - SYS_CHROOT
                - SETFCAP
            readOnlyRootFilesystem: true
          livenessProbe:
            httpGet:
              path: /healthz
              port: 9114
            initialDelaySeconds: 30
            timeoutSeconds: 10
          name: elastic-exporter
          ports:
            - containerPort: 9114
              name: http
          readinessProbe:
            httpGet:
              path: /healthz
              port: 9114
            initialDelaySeconds: 10
            timeoutSeconds: 10
          resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 25m
              memory: 64Mi
      restartPolicy: Always
      securityContext:
        runAsNonRoot: true
        runAsGroup: 10000
        runAsUser: 10000
        fsGroup: 10000
EOF
````

