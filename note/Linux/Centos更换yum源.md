# Centos更换yum源

## 备份原镜像

    mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup 
    
## 下载新的镜像

1.阿里云源

    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
    
    
2. 162镜像

    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS6-Base-163.repo

## 运行yum makecache 生成缓存

    yum makecache