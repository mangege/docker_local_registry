# Docker Local Registry


### Linux 下使用方法

    # 克隆代码
    git clone https://github.com/mangege/docker_local_registry.git
    # 切换目录
    cd docker_local_registry
    # 运行 Registry,可以把参数改成你自己的.
    docker run -d -p 5000:5000 --restart=always --name registry \
      -v `pwd`/registry-certs:/registry-certs \
      -v `pwd`/registry:/var/lib/registry \
      -e REGISTRY_HTTP_TLS_CERTIFICATE=/registry-certs/fullchain.pem \
      -e REGISTRY_HTTP_TLS_KEY=/registry-certs/privkey.pem \
      registry:2
    # 添加 10.10.10.10 IP 到本机现在使用的网卡,记得把 enp3s0 改成你的网卡标识. 重启后,每次都需要运行这个添加 IP 的命令.
    ip addr add 10.10.10.10/24 dev enp3s0
    # 测试是否能访问, 返回 {} 表示成功
    curl https://docker-registry.mangege.com:5000/v2/

    # 使用此 registry, 参考 https://github.com/docker/docker.github.io/blob/master/registry/deploying.md
    docker pull ubuntu && docker tag ubuntu docker-registry.mangege.com:5000/ubuntu
    docker push docker-registry.mangege.com:5000/ubuntu
    docker pull docker-registry.mangege.com:5000/ubuntu

### 背景

方便在本机用虚拟机测试 Docker 集群,比如 Docker Swarm 和 Kubernetes.

虽然通过改 DOCKER_OPTS 的加上 [--insecure-registry](https://github.com/docker/docker.github.io/blob/master/registry/insecure.md) 参数来实现使用 http registry,但觉得每台机器都要改,太麻烦了.

### 原理

1. 把自己的子域名 docker-registry.mangege.com 生成 Let's Encrypt 证书, 把生成的证书放在 registry-certs 目录提供下载公开使用.
2. 把 docker-registry.mangege.com 域名指向 10.10.10.10 这个 IP.
3. 为本机的局域网网卡添加 10.10.10.10 这第二个 IP ,这样就可以通过 https://docker-registry.mangege.com:5000/v2/ 访问你的本机 registry .


### 其它

1. 证书使用 [le-dns](https://github.com/xdtianyu/scripts/tree/master/le-dns) 脚本的域名记录验证方式生成的,所以支持内网 IP.
2. Mac 和 Win 都是支持单网卡多 IP 的,只是使用 boot2docker 记得把 registry 虚拟机的 5000 端口转发到本机.
3. 局域网其它机器需要使用此 registry, 添加 10.10.10.0/24 这个网段的第二个 ip 就可以了,比如可以使用 10.10.10.20 这个 IP.
4. 重启后,需要再使用 docker-registry.mangege.com ,记得为你的网卡添加 10.10.10.10 这个第二 IP.
