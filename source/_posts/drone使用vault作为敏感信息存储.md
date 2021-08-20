---
title: drone使用vault作为敏感信息存储
typora-copy-images-to: drone使用vault作为敏感信息存储
date: 2021-06-09 14:12:37
tags:
    - drone
    - vault
categories: drone
---

使用vault来存储secret，需要使用drone的drone-vault插件
docker-compose文件内容如下：
<!-- more -->
```yaml
version: "2"

networks:
  gitea:
    external: false

services:
  drone:
    image: drone/drone
    environment:
    - DRONE_GITEA_SERVER=http://1.2.3.4:18080  # gitea的地址
    - DRONE_GITEA_CLIENT_ID=ff1a8b12-d53d-40f6-8352-168e22a430d0 
    - DRONE_GITEA_CLIENT_SECRET=xQCLpsvFPhw7-FBQBTEbPtODvTBsD5pPO4n4f0q9oxA=
    - DRONE_RPC_SECRET=123456789 
    - DRONE_SERVER_HOST=1.2.3.4:10000 # 写外部能访问到的地址
    - DRONE_SERVER_PROTO=http 
    restart: always
    networks:
      - gitea
    volumes:
      - $HOME/data/drone:/data
    ports:
      - "10000:80"

  ssh_runner:
    image: drone/drone-runner-ssh
    environment:
    - DRONE_RPC_SECRET=123456789 
    - DRONE_RPC_HOST=drone:80
    - DRONE_RPC_PROTO=http 
    - DRONE_RUNNER_CAPACITY=3
    - DRONE_RUNNER_NAME=ssh_tx
    - DRONE_SECRET_PLUGIN_ENDPOINT=http://drone_vault:3000
    - DRONE_SECRET_PLUGIN_TOKEN=7890bcce69bb685a9a424767fe9d1be1
    networks:
      - gitea

  docker_runner:
    image: drone/drone-runner-docker
    environment:
    - DRONE_RPC_SECRET=123456789 
    - DRONE_RPC_HOST=drone:80
    - DRONE_RPC_PROTO=http 
    - DRONE_RUNNER_CAPACITY=2
    - DRONE_RUNNER_NAME=docker_tx
    - DRONE_SECRET_PLUGIN_ENDPOINT=http://drone_vault:3000
    - DRONE_SECRET_PLUGIN_TOKEN=7890bcce69bb685a9a424767fe9d1be1
    networks:
      - gitea
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock

  vault:
    image: vault:latest
    restart: always
    container_name: vault
    networks:
      - gitea
    volumes:
      - $HOME/data/vault/file:/vault/file
      - $HOME/data/vault/config:/vault/config
      - $HOME/data/vault/logs:/vault/logs
    cap_add:
      - IPC_LOCK
    environment:
      - VAULT_API_ADDR=http://127.0.0.1:8200
      - VAULT_ADDR=http://127.0.0.1:8200
      - VAULT_LOCAL_CONFIG={"backend":{"file":{"path":"/vault/file"}},"default_lease_ttl":"168h","max_lease_ttl":"720h","ui":true,"listener":{"tcp":{"address":"0.0.0.0:8200","tls_disable":1}}}
    ports:
      - 10001:8200
    command: vault server -config=/vault/config/local.json

  drone-vault:
    image: drone/vault
    container_name: drone_vault
    restart: always
    networks:
      - gitea
    environment:
      - DRONE_SECRET=7890bcce69bb685a9a424767fe9d1be1 #和ssh_runner以及docker_runner相同
      - DEBUG=true
      - VAULT_ADDR=http://vault:8200
      - VAULT_TOKEN_RENEWAL=84h
      - VAULT_TOKEN_TTL=168h
      - VAULT_TOKEN=s.CQye7biA8bvF8YybOIiaTZtF 
    ports:
      - 3000:3000
```
部署完成后，可以使用drone的cli来测试一下能不能取的到值，需要指定环境变量
```bash
export DRONE_SECRET_SECRET=7890bcce69bb685a9a424767fe9d1be1 # 和和ssh_runner以及docker_runner以及drone-vault相同
export DRONE_SECRET_ENDPOINT=http://127.0.0.1:3000 # 指定drone-vault对外映射的地址
```

设置完成后，即可使用命令
```bash
drone plugins secret get crawl/data/server user --repo octocat/hello-world
```
假如在vault中设定的kv secret是crawl/server，那么在drone cli中需要写crawl/data/server，在build.yml中也是同理。后面的user是crawl/server路径下的key名  --repo 随便指定即可

在build.yml中取vault的值采取以下形式
```yaml
---
kind: pipeline
type: ssh
name: deploy_to_server

trigger:
  branch:
  - master
  event:
  - push
  - custom

server:
  host: 
    from_secret: host
  user:
    from_secret: user
  password:
    from_secret: passwd

steps:
- name: submodules
  commands:
  - git submodule update --init --recursive

- name: greeting
  commands:
  - docker build -t test:test .
--- # 每取一个值都需要一个yaml的分隔符
kind: secret
name: host
get:
  path: crawl/data/server
  name: host
---
kind: secret
name: passwd
get:
  path: crawl/data/server
  name: password
---
kind: secret
name: user
get:
  path: crawl/data/server
  name: user
```

参考:
  - https://discourse.drone.io/t/drone-vault-plugin-not-quite-working/3641/12 官方人员的回答
  - https://learnku.com/articles/50294  参考的博客
  - https://docs.drone.io/runner/extensions/vault/ 官方文档