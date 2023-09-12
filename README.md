<p align="center"> 
 <img src="https://readme-typing-svg.herokuapp.com?color=%2336BCF7&center=true&vCenter=true&lines=FN+PROJECT" /> 
 </p> 
  
 ![Rerechan02 card name](https://cardivo.vercel.app/api?name=Rerechan02『𝐅𝐍』&description=Hi,%20everyone!%20and%20Nice%20to%20meet%20you%20%F0%9F%91%8B&image=https://raw.githubusercontent.com/Rerechan02/simple-xray/main/funny1.jpg?v=4&backgroundColor=%23ecf0f1&telegram=/&github=Rerechan02&pattern=leaf&colorPattern=%23eaeaea)


sslh -- A ssl/ssh multiplexer
=============================
`sslh` accepts connections on specified ports, and forwards
them further based on tests performed on the first data
packet sent by the remote client.

Probes for HTTP, TLS/SSL (including SNI and ALPN), SSH,
OpenVPN, tinc, XMPP, SOCKS5, are implemented, and any other
protocol that can be tested using a regular expression, can
be recognised. A typical use case is to allow serving
several services on port 443 (e.g. to connect to SSH from
inside a corporate firewall, which almost never block port
443) while still serving HTTPS on that port.

Hence `sslh` acts as a protocol demultiplexer, or a
switchboard. With the SNI and ALPN probe, it makes a good
front-end to a virtual host farm hosted behind a single IP
address.

`sslh` has the bells and whistles expected from a mature
daemon: privilege and capabilities dropping, inetd support,
systemd support, transparent proxying, chroot, logging,
IPv4 and IPv6, TCP and UDP, a fork-based and a select-based
model, and more.
=======


Configuration
=============



Docker image
------------

How to use

---


```bash
docker run \
  --cap-add CAP_NET_RAW \
  --cap-add CAP_NET_BIND_SERVICE \
  --rm \
  -it \
  ghcr.io/yrutschle/sslh:latest \
  --foreground \
  --listen=0.0.0.0:443 \
  --ssh=hostname:22 \
  --tls=hostname:443
```

docker-compose example

```yaml
version: "3"

services:
  sslh:
    image: sslh:latest
    hostname: sslh
    ports:
      - 443:443
    command: --foreground --listen=0.0.0.0:443 --tls=nginx:443 --openvpn=openvpn:1194
    depends_on:
      - nginx
      - openvpn

  nginx:
    image: nginx

  openvpn:
    image: openvpn
```

Transparent mode 1: using sslh container for networking

_Note: For transparent mode to work, the sslh container must be able to reach your services via **localhost**_
```yaml
version: "3"

services:
  sslh:
    build: https://github.com/yrutschle/sslh.git
    container_name: sslh
    environment:
      - TZ=${TZ}
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - NET_BIND_SERVICE
    sysctls:
      - net.ipv4.conf.default.route_localnet=1
      - net.ipv4.conf.all.route_localnet=1
    command: --transparent --foreground --listen=0.0.0.0:443 --tls=localhost:8443 --openvpn=localhost:1194
    ports:
      - 443:443 #sslh

      - 80:80 #nginx
      - 8443:8443 #nginx

      - 1194:1194 #openvpn
    extra_hosts:
      - localbox:host-gateway
    restart: unless-stopped

  nginx:
    image: nginx:latest
    .....
    network_mode: service:sslh #set nginx container to use sslh networking.
    # ^^^ This is required. This makes nginx reachable by sslh via localhost
  
  openvpn:
    image: openvpn:latest
    .....
    network_mode: service:sslh #set openvpn container to use sslh networking
```

Transparent mode 2: using host networking

```yaml
version: "3"

services:
  sslh:
    build: https://github.com/yrutschle/sslh.git
    container_name: sslh
    environment:
      - TZ=${TZ}
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - NET_BIND_SERVICE
    # must be set manually
    #sysctls:
    #  - net.ipv4.conf.default.route_localnet=1
    #  - net.ipv4.conf.all.route_localnet=1
    command: --transparent --foreground --listen=0.0.0.0:443 --tls=localhost:8443 --openvpn=localhost:1194
    network_mode: host
    restart: unless-stopped
  
  nginx:
    image: nginx:latest
    .....
    ports:
      - 8443:8443 # bind to docker host on port 8443

  openvpn:
    image: openvpn:latest
    .....
    ports:
      - 1194:1194 # bind to docker host on port 1194
```


## List of main arguments

| Name | Description | Values | Default |
| --- | --- | --- | --- |
| data | Dataset used | (sup|ssl)_(ads|cifar10|esc10|fsd50k|gsc|pvc|ubs8k) | (sup|ssl)_esc10 |
| pl | Pytorch Lightning training method (experiment) used | *(depends of the python script, see the filenames in config/pl/ folder)* | *(depends of the python script)* |
| model | Pytorch model to use | mobilenetv1, mobilenetv2, vgg, wideresnet28 | wideresnet28 |
| optim | Optimizer used | adam, sgd | adam |
| sched | Learning rate scheduler | cosine, softcosine, none | softcosine |
| epochs | Number of training epochs | int | 1 |
| bsize | Batch size in SUP methods | int | 60 |
| ratio | Ratio of the training data used in SUP methods | float in [0, 1] | 1.0 |
| bsize_s | Batch size of supervised part in SSL methods | int | 30 |
| bsize_u | Batch size of unsupervised part in SSL methods | int | 30 |
| ratio_s | Ratio of the supervised training data used in SSL methods | float in [0, 1] | 0.1 |
| ratio_u | Ratio of the unsupervised training data used in SSL methods | float in [0, 1] | 0.9 |


## SSLH Package overview
```
sslh
├── callbacks
├── datamodules
│     ├── supervised
│     └── semi_supervised
├── datasets
├── pl_modules
│     ├── deep_co_training
│     ├── fixmatch
│     ├── mean_teacher
│     ├── mixmatch
│     ├── mixup
│     ├── pseudo_labeling
│     ├── remixmatch
│     ├── supervised
│     └── uda
├── metrics
├── models
├── transforms
│     ├── get
│     ├── image
│     ├── other
│     ├── pools
│     ├── self_transforms
│     ├── spectrogram
│     └── waveform
└── utils
```

## Authors
This repository has been created by Etienne Labbé (Labbeti on Github).

It contains also some code from the following authors :
- Léo Cancès (leocances on github)
  - For AudioSet, ESC10, GSC, PVC and UBS8K datasets base code.
- Qiuqiang Kong (qiuqiangkong on Github)
  - For MobileNetV1 & V2 model implementation from [PANN](https://github.com/qiuqiangkong/audioset_tagging_cnn).

## Additional notes
- This project has been made with Ubuntu 20.04 and Python 3.8.5.

## Glossary
| Acronym | Description |
| --- | --- |
| activation | Activation function |
| ADS | AudioSet |
| aug, augm, augment | Augmentation |
| ce | Cross-Entropy |
| expt | Experiment |
| fm | FixMatch |
| fn, func | Function |
| GSC | Google Speech Commands dataset (with 35 classes) |
| GSC12 | Google Speech Commands dataset (with 10 classes from GSC, 1 unknown class and 1 silence class) |
| hparams | Hyperparameters |
| js | Jensen-Shannon |
| kl | Kullback-Leibler |
| loc | Localisation |
| lr | Learning Rate |
| mm | MixMatch |
| mse | Mean Squared Error |
| pred | Prediction |
| PVC | Primate Vocalize Corpus dataset |
| rmm | ReMixMatch |
| _s | Supervised |
| sched | Scheduler |
| SSL | Semi-Supervised Learning |
| SUP | Supervised Learning |
| _u | Unsupervised |
| UBS8K | UrbanSound8K dataset |

# sslh
Docker image exposing sslh (SSH/HTTPS/OpenVPN multiplexer) 

## Quick Start

* Start the SSLH server process, expose port 443

```
    docker run -d -p 443:443 --name sslh1 amondit/sslh
```

* You can set environment variables to define the service routing (here with their default values).

```
     LISTEN_IP 0.0.0.0
     LISTEN_PORT 443
     SSH_HOST localhost
     SSH_PORT 22
     OPENVPN_HOST localhost
     OPENVPN_PORT 1194
     HTTPS_HOST localhost
     HTTPS_PORT 8443
```

* However if you need to route to other Docker containers, don't forget to link them and then place the instance name in the environment:
 
```
    docker run -d -p 443:443 --link web1 -e HTTPS_HOST=web1 -e HTTPS_PORT=443  amondit/sslh
```

## Notes

Please note that I defaulted the HTTPS forward port to localhost:8443 because the service will by default bing sslh on all interfaces and setting the forward to localhost:443 could cause a loop. It's no big deal anyway as you will probably use this image to reference external services (whether they're linked Docker containers or external IPs), so you'll never have to use localhost. Unless you're using this image as a base for extension to run multiple services on 1 container, which you shouldn't as it would be against the philosophy of Docker.

<a name="提供的服务"></a>
提供的服务
-----------------

* [OpenSSH](https://www.openssh.com/)
  * 支持 Windows 和 Android 的 SSH 隧道， 并且需要使用 PuTTY 将默认的密钥对导出成 .ppk 的格式；
  * [Tinyproxy](https://banu.com/tinyproxy/) 默认安装并绑定到主机，它作为一个 http(s) 代理提供给那些原生不支持 SOCKS 代理的软件通过 SSH 隧道访问网络，比如说 Android 上的鸟嘀咕。
  * 针对 [sshuttle](https://github.com/sshuttle/sshuttle) 的一个无特权转发用户和产生的 SSH 密钥对，同样也兼容 SOCKS；
* [OpenConnect](https://ocserv.gitlab.io/www/index.html) / [Cisco AnyConnect](https://www.cisco.com/c/en/us/products/security/anyconnect-secure-mobility-client/index.html)
  * oepnConnect (ocserv) 是一个非常强劲、轻巧的 VPN 服务器，并且完全兼容思科的 AnyConnect 客户端；
  * 其中包涵了很多顶级的标准[协议](https://ocserv.gitlab.io/www/technical.html)，比如：HTTP, TLS 和 DTLS， 当然还有很多被跨国公司广泛使用的且流行的技术；
   * 这就意味着 OpenConnect 非常易用且高速，而且经得住审查的考验，几乎从未被封锁。
* [OpenVPN](https://openvpn.net/index.php/open-source.html)
  * 从自带的 .ovpn 配置文件生成一个简单的客户端配置文件；
  * 同时支持 TCP 和 UDP 连接；
  * 客户端的 DNS 解析由 [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) 负责，避免 DNS 泄露；
  * 启用 TLS 认证，有助于防止主动探测攻击。错误的 HMAC 流量并不会被轻易丢弃。
* [Shadowsocks](https://shadowsocks.org/en/index.html)
  * 安装的是高性能的 libev 版本，这个版本能够处理数以千计的并发连接；
  * Android 和 iOS 只需要扫描一个二维码就能完成自动配置。DNS 可以设置为 8.8.8.8，或者将配置一一复制粘贴到客户端上；
  * 采用 ChaCha20 和 Poly1305 对 [AEAD](https://shadowsocks.org/en/spec/AEAD-Ciphers.html) 进行加密，增强了安全性并提升了穿透性；
  * 使用 [simple-obfs](https://github.com/shadowsocks/simple-obfs) 插件提供流量混淆以便于从审查的网络中脱逃（尤其是QOS节流中）。
* [sslh](https://www.rutschle.net/tech/sslh/README.html)
  * sslh 是一个协议解复用器（这个我不了解，如果有更好的翻译请request），在一个高度限制的网络环境下（只能访问 http 端口的网络为例），它作为一种备选方案，仍然可以通过 OpenSSH 和 OpenVPN 进行连接，因为通过 sslh 让二者共享了 443 端口。
* [Stunnel](https://www.stunnel.org/index.html)
  * 监听并且将 OpenVPN 的流量进行封装，让 OpenVPN 的流量伪装成标注的 SSL 流量，从而可以让 OpenVPN 客户端成功通过隧道进行连接，躲避深度包检测。
  * 通过隧道连接的 OpenVPN 的配置文件和直接连接 OpenVPN 的配置文件是一起生成的，详细的说明也一同生成。
  * stunnel 证书和密钥是 PKCS #12 格式，SSL 隧道程序能够很好的兼容，尤其 [OpenVPN Android](https://play.google.com/store/apps/details?id=de.blinkt.openvpn) 版本也能够通过 [SSLDroid](https://play.google.com/store/apps/details?id=hu.blint.ssldroid) 传输。在中国的移动设备上使用 OpenVPN 成为可能*（据译者所知，大陆 OpenVPN 以前完全阵亡）*。
* [Tor](https://www.torproject.org/)
  * 桥接的名字是随即生成的；
  * [Obfsproxy](https://www.torproject.org/projects/obfsproxy.html.en) 默认安装并且配置支持 obfs4 传输；
  * Android 手机使用 [Orbot](https://play.google.com/store/apps/details?id=org.torproject.android) 扫描二维码即可获取桥接信息并完成自动配置。
* [UFW](https://wiki.ubuntu.com/UncomplicatedFirewall)
  * 防火墙根据不同的服务完全配置好，任何未经授权的流量都将被阻止。
* [系统无人值守安全更新](https://help.ubuntu.com/community/AutomaticSecurityUpdates)
  * Streisand 所在的服务器自动配置成无人值守更新，自动更新级别为*安全更新*。
* [WireGuard](https://www.wireguard.com/)
  * Linux 用户可以使用下一代更加简化，基于内核的黑科技 VPN ，速度快，而且使用了很多之前 VPN 所没有加密类型。
  
  <a id="cloud-providers"></a>
## Cloud providers
* Amazon Web Services (AWS)
* Microsoft Azure
* Digital Ocean
* Google Compute Engine (GCE)
* Linode
* Rackspace

### 重要说明 ###
Streisand 基于 [Ansible](https://www.ansible.com/) ，它可以在远程服务器完成自动配置、打包等工作，Streisand 是将远程服务器自动配置成为多个 VPN 服务及科学上网的工具。

Streisand 运行在**你自己的计算机上时（或者你电脑的虚拟机上时）**，它将把网关部署到你 VPS 提供商的**另一个服务器**上（通过你自己的API自动生成）。另外，如果 Streisand 运行在 VPS 上，它将会把网关部署到**另一个 VPS 上**，所以说原先你运行 Streisand 的那个 VPS 就多余了，记得部署完成并获得文档后把它删掉，而部署出来的那个 VPS 你是无法使用 SSH 连接进去的，除非你有公钥（当然这是不可能的，因为整个配置过程都没有提供公钥给你下载或者你想办法把它搞出来）。

在某些情况下，你可能需要运行 Streisand/Ansible 在 VPS 上并将其自身配置为 Streisand 服务器，这种模式适合当你无法在你自己的计算机上运行和安装 Streisand/Ansible 时，或本地与 VPS 之间的 SSH 连接不稳定时。

### 准备工作 ###
在本地计算机完成以下所有步骤（也可以在 VPS 上运行）。

* Streisand 运行在 BSD，Linux，或者 macOS 上，Windows 上是无法运行的。所有的指令都需要在终端下运行。
* 需要 Python 2.7 ，一般在 macOS 、Linux 及 BSD 系统上都是标配，如果你使用的发行版标配是 Python 3，你需要安装 Python 2.7 来运行 Streisand。
* 确定你的 SSH key 储存在 ~/.ssh/id\_rsa.pub 。
  * 如果不曾有过 SSH key，你需要用下面的命令生成一个：

        ssh-keygen
* 安装 Git 。
  * 基于 Debian 和 Ubuntu 的 Linux 发行版

        sudo apt install git
  * 在 Fedora

        sudo dnf install git
  * 在 macOS 上 （需要通过 [Homebrew](https://brew.sh/) 进行安装）

        brew install git
* 利用 Python 安装 [pip](https://pip.pypa.io/en/latest/) 包管理
  * 基于 Debian 和 Ubuntu 的 Linux 发行版

        sudo apt install python-paramiko python-pip python-pycurl python-dev build-essential
  * 在 Fedora

        sudo dnf install python-pip
  * 在 macOS 上

        sudo easy_install pip
        sudo pip install pycurl

* 安装 [Ansible](https://www.ansible.com/) 。
  * 在 macOS 上

        brew install ansible
  * 在 Linux 和 其他 BSD 上

        sudo pip install ansible markupsafe
* 以下使用 pip 安装的 Python 库根据你所使用的 VPS 供应商不同而不同。如果你准备将目前使用的 VPS 变成网关，可以跳过此步。
  * 亚马逊 EC2

        sudo pip install boto boto3
  * 微软云服务

        sudo pip install ansible[azure]
  * Google

        sudo pip install "apache-libcloud>=1.17.0"
  * Linode

        sudo pip install linode-api4
  * Rackspace 云

        sudo pip install pyrax
  * **特别需要注意的是如果你使用 Python** 是通过 Homebrew 安装的，还需要运行以下命令来确定找到必要的库文件

        mkdir -p ~/Library/Python/2.7/lib/python/site-packages
        echo '/usr/local/lib/python2.7/site-packages' > ~/Library/Python/2.7/lib/python/site-packages/homebrew.pth

### 执行 ###
1. 从 Streisand 抓取源码

        git clone https://github.com/StreisandEffect/streisand.git && cd streisand
2. 执行 Streisand 脚本。

        ./streisand
3. 根据实际情况从弹出的问题中填写或选择选项，比如服务器的物理位置，它的名字。还有最重要的是 API 信息（可以从问题提示中找到如何提供 API 信息）。
4. 一旦登录信息和 API key 正确无误完成，Streisand 就开始安装到另一个 VPS 上了（或者把你目前的 VPS 变成网关）。
5. 整个配置完成大概需要10分钟左右的样子，完成后，在 Streisand 目录下会生成一个 'generated-docs' 文件夹，里面储存了4个 html 文件，其中包含了网关的 SSL 证书和如何连接的详细说明。当你使用这些方法连接上网关以后，网关里详细描述了设置客户端的方法、需要额外下载的文件，客户端镜像，密钥，只要耐心配置客户端就一切搞定了。

**译者注：到这里官方英文配置就告一段落了。译者在自己配置的过程中还遇到一些小麻烦，需要各位朋友注意。**
* 从本地用 Streisand 安装到网关的模式，如果能用这种模式最好，就不要选择其他的模式了，因为这样他生成的 generated-docs 就在本地，你用浏览器打开就能直接下载到证书文件，不折腾。
* 在 VPS 上用 Streisand 安装到新的 VPS 模式还有后面介绍的将正在运行的 VPS 转变为网关这种模式，你会发现你很难在 VPS 上阅读 generated-docs 文件夹中的4个 html 文档，这个时候有几种方法可选：
  * 使用 sftp 下载文件；
  * 在目前的 VPS 上安装一个 apache2 ，然后 cp -r generated-doc /var/www/html/ ，然后从浏览器输入 VPS 的地址直接浏览并下载密钥（严格地说，这不安全，因为不是 https 连接，如果截获了数据便可以知道如何登录你科学上网的那个网关了。**如果是使用转化那个模式，就不要用这种方法了**）。
  * 在 VPS 上使用 scp 将 generated-docs 目录全部推送到你本地暴露在公网下的 Linux, Unix 或者 macOS 里，或者另一个 VPS 里也可以。命令大概是 scp -r generated-docs user@×××.×××.×××.×××:/home/user/

### 将正在使用中的 VPS 变成 Streisand 服务器 （高级使用） ###

如果你本地使用的计算机无法运行 Streisand ，你可以将正在运行的 VPS 转变为网关。只需要在 VPS 上运行 ./streisand 并在菜单中选择 "Localhost (Advanced)" 就可以了。

**但是需要注意的是**这个操作是无法挽回的，它将把你正在使用的 VPS 完全转变为网关后，之前如果你在上面搭建博客或者用于测试某些软件，那完成这个操作后，它们将不复存在。

### 在其他的 VPS 供应商上运行 （高级使用）###

你同样可以将 Streisand 运行在其他 VPS 供应商（提供更好的硬件也没问题，奇葩的 VPS 供应商也行）的 16.04 Ubuntu 上，只需要你在运行 ./streisand 的时候选择菜单中的 "Existing Server (Advanced)" 就可以。你需要提供这个 VPS 的 IP 地址。

这个 VPS 必须使用 `$HOME/.ssh/id_rsa` 来储存 SSH key，并且可以使用 **root** 作为默认用户登录 VPS，如果提供商没有给你 root 用户作为默认用户登录，而是别的用户名，比如：`ubuntu` ，那么在运行 `./streisand` 之前需要额外配置 `ANSIBLE_SSH_USER` 环境变量，比如修改为：`ANSIBLE_SSH_USER=ubuntu` 。

### 非交互式部署 （高级使用）###

如果你想做非交互式部署, 你可以在 `global_vars/noninteractive`找到配置文件和脚本文件。你需要在配置文件或命令行录入必要信息。

将 Streisand 在 VPS 供应商上运行:

      deploy/streisand-new-cloud-server.sh \
        --provider digitalocean \
        --site-config global_vars/noninteractive/digitalocean-site.yml

将 Streisand 在正在使用中的服务器上运行:

      deploy/streisand-local.sh \
        --site-config global_vars/noninteractive/local-site.yml

将 Streisand 在已现有的服务器上运行 :

      deploy/streisand-existing-cloud-server.sh \
        --ip-address 10.10.10.10 \
        --ssh-user root \
        --site-config global_vars/noninteractive/digitalocean-site.yml

未来特性
-----------------
* 更简便的设置

如果你对 Streisand 有任何期待和想法，或者你找到 BUG ，请联系我们并且发 [Issue Tracker](https://Rerechan02/fn_project) 。


### FLAKE8 ###
[flake8]
max-line-length = 88
extend-ignore = W191,E203,E501,E402
show_source = True
exclude =
    # No need to traverse our git directory
    .git,
    # There's no value in checking cache directories
    __pycache__,
    # The conf file is mostly autogenerated, ignore it
    docs/source/conf.py,
    # The old directory contains Flake8 2.0
    old,
    # This contains our built documentation
    build,
    # This contains builds of flake8 that we don't want to check
    dist,
    # Ignore notebook checkpoints
    .ipynb_checkpoints
per-file-ignores =
    # imported but unused
    __init__.py: F401

# Advanced installation

### Running Streisand to Provision Localhost ###

If you can't run Streisand in the normal manner (running from your client home machine/laptop to configure a remote server) Streisand supports a local provisioning mode for **machines running Ubuntu 16.04**. After following the basic instructions, choose "Localhost (Advanced)" from the menu after running `./streisand`.

**Note:** Running Streisand against localhost can be a destructive action! You will be potentially overwriting configuration files and must be certain that you are affecting the correct machine.

**Note:** The machine must be running Ubuntu 16.04.

**Note:** If you have an external firewall available, see the `generated-docs` directory for a `-firewall-information.html` document with information on which ports need to be opened.

### Running Streisand on Other Providers ###

You can also run Streisand on a new Ubuntu 16.04 server. Dedicated hardware? Great! Esoteric cloud provider? Awesome! To do so, simply choose "Existing Server (Advanced)" from the menu after running `./streisand` and provide the IP address of the existing server when prompted.

The server must be accessible using the `$HOME/.ssh/id_rsa` SSH Key, and **root** is used as the connecting user by default. If your provider requires you to SSH with a different user than root (e.g. `ubuntu`) specify the `ANSIBLE_SSH_USER` environmental variable (e.g. `ANSIBLE_SSH_USER=ubuntu`) when you run `./streisand`.

**Note:** Running Streisand against an existing server can be a destructive action! You will be potentially overwriting configuration files and must be certain that you are affecting the correct machine.

**Note:** The machine must be running Ubuntu 16.04.

**Note:** If you have an external firewall available, see the `generated-docs` directory for a `-firewall-information.html` document with information on which ports need to be opened.

### Noninteractive Deployment ###

Alternative scripts and configuration file examples are provided for
noninteractive deployment, in which all of the required information is passed
on the command line or in a configuration file.

Example configuration files are found under `global_vars/noninteractive`. Copy
and edit the desired parameters, such as providing API tokens and other choices,
and then run the appropriate script.

To deploy a new Streisand server:

      deploy/streisand-new-cloud-server.sh \
        --provider digitalocean \
        --site-config global_vars/noninteractive/digitalocean-site.yml

To run the Streisand provisioning on the local machine:

      deploy/streisand-local.sh \
        --site-config global_vars/noninteractive/local-site.yml

To run the Streisand provisioning against an existing server:

      deploy/streisand-existing-cloud-server.sh \
        --ip-address 10.10.10.10 \
        --ssh-user root \
        --site-config global_vars/noninteractive/digitalocean-site.yml

# Features

* A single command sets up a brand new Ubuntu 16.04 server running a [wide variety of anti-censorship software](#services-provided) that can completely mask and encrypt all of your Internet traffic.
* Streisand natively supports the creation of new servers at [Amazon EC2](https://aws.amazon.com/ec2/), [Azure](https://azure.microsoft.com), [DigitalOcean](https://www.digitalocean.com/), [Google Compute Engine](https://cloud.google.com/compute/), [Linode](https://www.linode.com/), and [Rackspace](https://www.rackspace.com/)&mdash;with more providers coming soon! It also runs on any Ubuntu 16.04 server regardless of provider, and **hundreds** of instances can be configured simultaneously using this method.
* The process is completely automated and only takes about ten minutes, which is pretty awesome when you consider that it would require the average system administrator several days of frustration to set up even a small subset of what Streisand offers in its out-of-the-box configuration.
* Once your Streisand server is running, you can give the custom connection instructions to friends, family members, and fellow activists. The connection instructions contain an embedded copy of the server's unique SSL certificate, so you only have to send them a single file.
* Each server is entirely self-contained and comes with absolutely everything that users need to get started, including cryptographically verified mirrors of all common clients. This renders any attempted censorship of default download locations completely ineffective.
* But wait, there's more...

More Features
-------------
* Nginx powers a password-protected and encrypted Gateway that serves as the starting point for new users. The Gateway is accessible over SSL, or as a Tor [hidden service](https://www.torproject.org/docs/hidden-services.html.en).
  * Beautiful, custom, step-by-step client configuration instructions are generated for each new server that Streisand creates. Users can quickly access these instructions through any web browser. The instructions are responsive and look fantastic on mobile phones.
  * The integrity of mirrored software is ensured using SHA-256 checksums, or by verifying GPG signatures if the project provides them. This protects users from downloading corrupted files.
  * All ancillary files, such as OpenVPN configuration profiles, are also available via the Gateway.
  * Current Tor users can take advantage of the additional services Streisand sets up in order to transfer large files or to handle other traffic (e.g. BitTorrent) that isn't appropriate for the Tor network.
  * A unique password, SSL certificate, and SSL private key are generated for each Streisand Gateway. The Gateway instructions and certificate are transferred via SSH at the conclusion of Streisand's execution.
* Distinct services and multiple daemons provide an enormous amount of flexibility. If one connection method gets blocked there are numerous options available, most of which are resistant to Deep Packet Inspection.
  * All of the connection methods (including direct OpenVPN connections) are effective against the type of blocking Turkey has been experimenting with.
  * OpenConnect/AnyConnect, OpenSSH, OpenVPN (wrapped in stunnel), Shadowsocks, Tor (with obfsproxy and the obfs4 pluggable transport), and WireGuard are all currently effective against China's Great Firewall.
* Every task has been thoroughly documented and given a detailed description. Streisand is simultaneously the most complete HOWTO in existence for the setup of all of the software it installs, and also the antidote for ever having to do any of this by hand again.
* All software runs on ports that have been deliberately chosen to make simplistic port blocking unrealistic without causing massive collateral damage. OpenVPN, for example, does not run on its default port of 1194, but instead uses port 636, the standard port for LDAP/SSL connections that are beloved by companies worldwide.


# Installation

Please read all installation instructions **carefully** before proceeding.

If you're an expert, and installing on a cloud provider Streisand doesn't support, make sure to read [the advanced installation instructions](Advanced%20installation.md).

## Important: definitions ##
Streisand is based on [Ansible](https://www.ansible.com/), an automation tool that is typically used to provision and configure files and packages on remote servers. Streisand automatically sets up **another server** with the VPN packages and configuration. We call the machine that sets up the Streisand server the *builder*. Think of the builder as "a place to stand."

* If you don't have a suitable builder machine, you could set up another cloud server to use as your builder. That means you'd have two cloud servers at the end — the builder, and your fresh new Streisand *server*.  When you're done with the builder, make sure you download the *builder's* `streisand` directory — very important to keep the contents of that directory! — you could delete the *builder* cloud server.)

* Although it's not recommended, sometimes you can use a fresh server as both the builder and the server. See the [the advanced installation instructions](Advanced%20installation.md).

## Prerequisites ##

The Streisand builder requires a Linux, macOS, or BSD system.

* Using native Windows as a builder is not supported, but Ubuntu on the [Windows Subsystem For Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/faq) should work. ([Ubuntu install link on Microsoft Store](https://www.microsoft.com/en-us/p/ubuntu-1804-lts/9n9tngvndl3q))

Complete all of these tasks on your local machine. All of the commands should be run inside a command-line session.

### SSH key

Make sure an SSH public key is present in `~/.ssh/id_rsa.pub`.

  * SSH keys are a more secure alternative to passwords that allow you to prove your identity to a server or service built on public key cryptography. The public key is something that you can give to others, whereas the private key should be kept secret (like a password).

To check if you already have an SSH public key, enter the following command at a command prompt:

```
ls ~/.ssh
```

If you see an `id_rsa.pub` file, then you have an SSH public key. If you do not have an SSH key pair, you can generate one by using this command and following the defaults:

```
ssh-keygen
```

If you'd like to use an SSH key with a different name or from a non-standard location, please enter *yes* when asked if you'd like to customize your instance during installation.

  * **Please note**: You will need these keys to access your Streisand instance over SSH. Please keep `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` for the lifetime of the Streisand server.


## Bootstrap ##

Install the bootstrap packages: Git, and Python 3.5 or later. Some environments need additional packages.

Here's how to set up these packages:

* On Debian and Ubuntu:

```
sudo apt-get install git python3 python3-venv
```

* On Fedora 30, some additional packages are needed later:

```
dnf install git python3 gcc python3-devel python3-crypto \
     python3-pycurl libcurl-devel

```

* On CentOS 7, `python36` is available from the EPEL repository; some additional packages are needed later:

```
sudo yum -y update && sudo yum install -y epel-release
sudo yum -y update && sudo yum install -y \
    git gcc python36-devel python36-crypto python36-pycurl \
    libcurl-devel
```

* On macOS, `git` is part of the Developer Tools, and it will be installed the first time you run it. Python 3.5 or later is required. Either use [Homebrew](https://brew.sh/) and run `brew install python3`, or see [the Python.org Mac download site](https://www.python.org/downloads/mac-osx/); the package you want to download is the "macOS 64-bit installer".

## Execution ##

1. Clone the Streisand repository and enter the directory.

        git clone https://github.com/StreisandEffect/streisand.git && cd streisand

1. Run the installer for Ansible and its dependencies. The installer will detect missing packages, and print the commands needed to install them. (Ignore the Python 2.7 `DEPRECATION` warning; ignore the warning from python-novaclient that pbr 5.1.3 is incompatible.)

       ./util/venv-dependencies.sh ./venv

1. Activate the Ansible packages that were installed.

        source ./venv/bin/activate

1. Execute the Streisand script.

        ./streisand

1. Follow the prompts to choose your provider, the physical region for the server, and its name. You will also be asked to enter API information.
1. Once login information and API keys are entered, Streisand will begin spinning up a new remote server.
1. Wait for the setup to complete (this usually takes around ten minutes) and look for the corresponding files in the `generated-docs` folder in the Streisand repository directory. The HTML file will explain how to connect to the Gateway over SSL, or via the Tor hidden service. All instructions, files, mirrored clients, and keys for the new server can then be found on the Gateway. You are all done!

## Keep the results!

You should keep a copy of the `generated-docs` directory for the life of the server.

Remember to save your `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` SSH keys too. You'll need them in case you want to troubleshoot or perform maintenance on your server later.

## Contact
Maintainer:
- Rerechan02 "Rerechan02": widyabakti02@gmail.com