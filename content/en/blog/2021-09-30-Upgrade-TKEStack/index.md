---
layout: blog
title: "Upgrade TKEStack with tke-installer"
date: 2021-09-30
# slug of this blog url
slug: upgrade-tkestack
---

**Author**: LeoRyu

_This paper will introduce how to upgrade TKEStack with tke-installer._

## Limitations

1. TKEStack version >= 1.5.0
2. Cannot skip minor version, e.g., cannot upgrade from v1.5.x to v1.7.x
3. make sure your global cluster node have enough disk space, > 40GB is recommended
4. If upgrade from v1.7.x to v1.8.x, you should migrate container runtime from docker to containerd before upgrade, check https://tkestack.github.io/web/zh/blog/2021/09/01/container-runtime-migraion/


## Upgrade Process

### Download target version tke-installer

Download tke-installer with version you want to upgrade to in `global` cluster node.

```sh
arch=amd64 version={version you want to upgrade to} && wget https://tke-release-1251707795.cos.ap-guangzhou.myqcloud.com/tke-installer-linux-$arch-$version.run{,.sha256} && sha256sum --check --status tke-installer-linux-$arch-$version.run.sha256 && chmod +x tke-installer-linux-$arch-$version.run
```

### Prepare credential

```sh
# set kubeconfig
mkdir -p /opt/tke-installer/conf  && mkdir -p /opt/tke-installer/data && cp ~/.kube/config /opt/tke-installer/conf/kubeconfig

# set registry ca cert
kubectl exec -n tke tke-registry-api-{your registry api hash} cat certs/ca.crt > /opt/tke-installer/data/ca.crt
```

### Upgrade through tke-installer

```sh
./tke-installer-linux-amd64-{version you want to upgrade to}.run --upgrade
```

### Check upgrade log

```sh
tail -f /opt/tke-installer/data/tke.log
```

This log file will log upgrade process until you get `===>upgrade task [Sucesss]`.