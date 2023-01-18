# Windows 安装 MicroK8s 

MicroK8s 是 CNCF 认证的上游 Kubernetes 部署工具，可以在个人 PC 上快速构建 Kubernetes 环境。它可以在本地运行所有 Kubernetes 服务，并且删除 MicroK8s 不会留下任何东西。

我的环境 Win10 专业版（22H2），并且需要预先开启 Hype-v 功能和 容器 功能。开启方式参考以下步骤：
1. 打开“控制面板”选择“程序”
2. 选择“启用或关闭 Windows 功能”
3. 选择“Hyper-V”，然后单击“确定”
4. 重新启动电脑

安装主要步骤参考[官方文档 ](https://discourse.ubuntu.com/t/install-microk8s-on-windows/13967) 

# 如何使用 MicroK8s 部署应用

在部署应用之前，还需要对 Kubernetes 做一些准备工作。因为 Microk8s 部署的 Kubernetes 没有下载 pause 镜像，所以在创建 pod 时会报错，pod 一直处于 containercreating 状态，解决办法就是将 pause 镜像手动导入 kubernetes 。请参照以下步骤：

进入虚拟机（感觉在虚拟机中执行命令比较简单，不必用 microk8s 前缀调用命令了）在命令行中执行：
```
multipass shell microk8s-vm
```

下载并导入 pause 镜像：
```
sudo ctr i pull docker.io/mirrorgooglecontainers/pause:3.1
sudo ctr i tag docker.io/mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
sudo ctr i export k8s.gcr.io/pause:3.1 pause.tar
```

使用 Microk8s 的管理命令导入镜像（ Microk8s 有自己的镜像存放位置）：
```
sudo microk8s.ctr image import pause.tar
```

接下来就可以在 Kubernetes 中部署应用了。登录到 MicroK8s 控制器。在本演示中，我们将部署一个 NGINX Web 服务器应用程序。我们将此部署命名为 nginx-webserver，并使用官方 NGINX 容器映像进行部署。

这里附上一个命令给 microk8s kubectl 改别名：
```
sudo snap alias microk8s.kubectl kubectl
```

部署应用：
```
​microk8s kubectl create deployment nginx-webserver --image=nginx
```

输出结果如下所示则表示创建成功：

```
deployment.apps/nginx-webserver created
```

可以使用如下命令查看部署是否成功：

```
microk8s kubectl get pods
```

你应该看到如下结果：

```
nginx-webserver-67f557b648-4mfc6     1/1   Running
```

部署 service 以确保 Nginx 可以被访问：

```
microk8s kubectl expose deployment nginx-webserver --type="NodePort" --port 80
```

你应该看到如下结果：

```
service/webserver exposed
```

使用如下命令查看 service 信息：

```
microk8s kubectl get svc nginx-webserver
```

结果如下：

```
nginx-webserver  NodePort  10.152.183.105  &lt;none&gt;    80:31508/TCP  3m11s
```

使用如下命令查看节点IP地址：
```
microk8s kubectl get nodes -o wide
```

至此，你的第一个应用就部署成功了，在浏览器输入 http://172.31.18.101:31508 就可以访问到你的 Nginx 主页了！！！
