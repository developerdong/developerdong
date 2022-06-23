# k8s使用代理拉取镜像

如果你有一个代理，那么在执行`kubeadm init`的时候可以使用代理来拉取镜像。

一开始我将`HTTP_PROXY`和`HTTPS_PROXY`环境变量配置到docker.service中，
然后用`docker pull`命令进行验证，可以正常拉取k8s.gcr.io上的镜像。
配置方法可以参考[这里](https://blog.csdn.net/fjh1997/article/details/100829519)。

但是执行`kubeadm init`的时候，拉取镜像还是失败。
研究之后发现kubeadm是调用containerd去拉取镜像的，
所以设置代理的环境变量必须配置到containerd.service中才能生效。
systemd service中环境变量的配置方法参考[这里](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Environment)。
