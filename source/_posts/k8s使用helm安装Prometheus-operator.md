---
title: k8s使用helm安装Prometheus-operator
date: 2021-08-23 10:09:35
tags:
	- k8s
    - prometheus
categories: k8s
---
使用helm来安装以来，首先安装helm
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```
<!-- more -->
然后使用helm安装对应的repo，对应的chart链接为:https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

然后安装对应的chart，使用下面的命令
```bash
 helm install my-kube-prometheus-stack --namespace=monitoring prometheus-community/kube-prometheus-stack --version 18.0.0 --set prometheusOperator.admissionWebhooks.enabled=false --set prometheusOperator.admissionWebhooks.patch.enabled=false --set prometheusOperator.tls.enabled=false
```

如果安装失败时，需要uninstall，也需要在helm对应的namespace下
```bash
helm uninstall xxx -n monitoring
```

相关的页面，有错时可以参考:
https://github.com/prometheus-community/helm-charts/issues/418
https://github.com/helm/charts/issues/21080#issuecomment-596958610
