## 1、官网脚本

```shell  
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

!!! tip "不推荐"
    由于国内访问 http://raw.githubusercontent.com 不稳定，很可能下载很慢，甚至失败。


## 2、镜像脚本

```shell
  /usr/bin/ruby -e "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install)"
```

!!! tip "推荐"
    这是中科大的镜像脚本，速度很快。




!!! warning "环境变量"
    安装完 Homebrew 时，如果需要设置环境变量会有相应的log，在log中复制其命令执行即可。