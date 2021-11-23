---
title: k8s从文件生成configmap的问题
date: 2021-11-23 17:39:35
tags:
	- k8s
    - configmap
categories: k8s
---
使用k8s的kustomize工具，生成configmap时，或者直接从file创建configmap时，有时会遇到一个问题，
就是configmap中对应文件的内容直接被解析为一个大的字符串，换行符都变为了\n的显示，
```yaml
data:
    key: "header\n11"
```
而我们期望的格式是
```yaml
data:
    key: |-
    header
    11
```

参考此链接的解决方法，https://stackoverflow.com/questions/51291521/kubernetes-configmap-prints-n-instead-of-a-newline
将文件中的tab换成空格，并去除掉行尾多余的空格即可。
使用以下命令
```bash
sed -i -E 's/[[:space:]]+$//g' collect_disk_fault.sh
sed -i 's/\t/    /g' collect_disk_fault.sh
```
再次生成configmap即可得到我们想要的格式