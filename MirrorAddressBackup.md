# 配置国内镜像 - apt/brew/Docker/Python Conda/Maven


## 此处仅列出了常用的一些，还有很多没有列出来，需要的话可以从文章最下方链接中查找相应的国内镜像
## Ubuntu
### apt
vim /etc/apt/sources.list

## 清华源

### 16.04 LTS
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```

### 18.04 LTS
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

### 18.10
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-proposed main restricted universe multiverse
```

### 其他版本详见 https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/

### 中科大源 https://mirrors.ustc.edu.cn/help/ubuntu.html

### 阿里源 https://opsx.alibaba.com/mirror 找到 Ubuntu，点击帮助

### 网易源 https://mirrors.163.com/.help/ubuntu.html 不全

## CentOS
### yum
/etc/yum.repos.d/CentOS-Base.repo

### 清华源

### CentOS 7
```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### CentOS 6
```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

### CentOS 5
```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#packages used/produced in the build but not released
[addons]
name=CentOS-$releasever - Addons
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/addons/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=addons
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
```

### 清华源 https://mirror.tuna.tsinghua.edu.cn/help/centos/

### 中科大 https://mirrors.ustc.edu.cn/help/centos.html

### 阿里源 https://opsx.alibaba.com/mirror

## Docker
### 安装
### 清华源提供 Debian/Ubuntu/Fedora/CentOS/RHEL 的 docker 软件包

### Ubuntu
```
sudo apt-get remove docker docker-engine docker.io
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce
```
### 其他看 https://mirror.tuna.tsinghua.edu.cn/help/docker-ce/

### 中科大 https://mirrors.ustc.edu.cn/help/docker-ce.html

### Docker Hub
Docker 中国 https://www.docker-cn.com/registry-mirror

### 中科大
https://mirrors.ustc.edu.cn/help/dockerhub.html

### 国内 docker 仓库镜像对比
https://link.zhihu.com/?target=https%3A//ieevee.com/tech/2016/09/28/docker-mirror.html

### Linux
对于使用 upstart 的系统（Ubuntu 14.04、Debian 7 Wheezy），在配置文件 /etc/default/docker 中的 DOCKER_OPTS 中配置Hub地址：
` DOCKER_OPTS="--registry-mirror=https://docker.mirrors.ustc.edu.cn/" `
重新启动服务:
`sudo service docker restart`
对于使用 systemd 的系统（Ubuntu 16.04+、Debian 8+、CentOS 7）， 在配置文件 /etc/docker/daemon.json 中加入：
```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
```
重新启动 dockerd：
```
sudo systemctl restart docker
```

### macOS
打开 “Docker.app” 进入偏好设置页面(快捷键 ⌘, ) 打开 “Daemon” 选项卡 在 “Registry mirrors” 中添加 https://docker.mirrors.ustc.edu.cn/ 点击下方的 “Apply & Restart” 按钮

### Windows
在系统右下角托盘 Docker 图标内右键菜单选择 Settings ，打开配置窗口后左侧导航菜单选择 Daemon 。在 Registry mirrors 一栏中填写地址 https://docker.mirrors.ustc.edu.cn/ ，之后点击 Apply 保存后 Docker 就会重启并应用配置的镜像地址了。

检查 Docker Hub 是否生效 在命令行执行 docker info ，如果从结果中看到了如下内容，说明配置成功。
```
Registry Mirrors:
    https://docker.mirrors.ustc.edu.cn/
```    

### Anaconda
### 下载安装
#### 清华 https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/

#### 中科大 https://mirrors.ustc.edu.cn/anaconda/archive/

### Conda
https://mirror.tuna.tsinghua.edu.cn/help/anaconda/

https://mirrors.ustc.edu.cn/help/anaconda.html
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```

### Homebrew
### 清华

https://mirror.tuna.tsinghua.edu.cn/help/homebrew/

替换现有上游
```
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

brew update
```
使用homebrew-science或者homebrew-python
```
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-science"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-science.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-python"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-python.git

brew update
```
https://mirror.tuna.tsinghua.edu.cn/help/homebrew-bottles/

该镜像是 Homebrew 二进制预编译包的镜像。本镜像站同时提供 Homebrew 的 formula 索引的镜像（即 brew update 时所更新内容）

临时替换
```
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
```

长期替换
```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

中科大
```
https://mirrors.ustc.edu.cn/help/brew.git.html

https://mirrors.ustc.edu.cn/help/homebrew-bottles.html

https://mirrors.ustc.edu.cn/help/homebrew-core.git.html

https://mirrors.ustc.edu.cn/help/homebrew-cask.git.html
```

### Maven
~/.m2/settings.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
          <mirrors>
            <mirror>
              <id>alimaven</id>
                <name>aliyun maven</name>
                <url>https://maven.aliyun.com/repository/central</url>
                <mirrorOf>central</mirrorOf>
              </mirror>
          </mirrors>
</settings>
```


## 镜像站汇总
清华大学开源软件镜像站 - Tsinghua Open Source Mirror
https://link.zhihu.com/?target=https%3A//mirrors.tuna.tsinghua.edu.cn/

中国科学技术大学开源软件镜像
https://link.zhihu.com/?target=http%3A//mirrors.ustc.edu.cn/

淘宝 NPM 镜像
https://link.zhihu.com/?target=https%3A//npm.taobao.org/

网易开源镜像站
https://link.zhihu.com/?target=http%3A//mirrors.163.com/

阿里巴巴开源镜像站
https://link.zhihu.com/?target=https%3A//opsx.alibaba.com/

搜狐开源镜像站
https://link.zhihu.com/?target=http%3A//mirrors.sohu.com/

Capital Online Mirror Site
https://link.zhihu.com/?target=http%3A//mirrors.yun-idc.com/

北京理工大学开源软件镜像站
https://link.zhihu.com/?target=http%3A//mirror.bit.edu.cn/web/

北京交通大学开源软件镜像站
https://link.zhihu.com/?target=https%3A//mirror.bjtu.edu.cn/cn/

兰州大学开源社区镜像站
https://link.zhihu.com/?target=http%3A//mirror.lzu.edu.cn/

浙江大学开源镜像站
https://link.zhihu.com/?target=http%3A//mirrors.zju.edu.cn/

Northeastern University OpenSource Mirrors
https://link.zhihu.com/?target=http%3A//mirror.neu.edu.cn/

华中科技大学开源镜像站
https://link.zhihu.com/?target=http%3A//mirrors.hust.edu.cn/

重庆大学开源软件镜像站
https://link.zhihu.com/?target=http%3A//mirrors.cqu.edu.cn/

东软信息学院自己的开源镜像站
https://link.zhihu.com/?target=http%3A//mirrors.neusoft.edu.cn/

编辑于 2019-03-20
