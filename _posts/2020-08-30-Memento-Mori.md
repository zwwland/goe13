
[TOC]

### 自动登陆脚本
```bash
#!/usr/bin/expect
set timeout 30
spawn ssh -p [lindex $argv 0] [lindex $argv 1]@[lindex $argv 2]
expect {
        "*(yes/no)?"
        {send "yes\n";exp_continue}
        "*assword:"
        {send "[lindex $argv 3]\n"}
}
interact
```

### ssh 免密登陆
1. 生成密钥对
```bash
ssh-keygen -t rsa -C m4a1player@126.com
```
2. 服务器添加免密文件
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@1.1.1.1
or
cat ~/.ssh/id_rsa.pub >> root@1.1.1.1:~/.ssh/authorized_keys
```
3. 本机配置
```bash
nvim ~/.ssh/config
Host aaaa
    > HostName 1.1.1.1
    > User root
    > Port 22
    > IdentityFile ~/.ssh/id_rsa
```
4. ssh aaaa 登陆
5. 重设ssh私钥密码
```bash
ssh-keygen -f id_rsa -p
```

### ">" "/dev/null" "0,1,2" 说明
```
/dev/null 空设备文件
> 输出重定向
0 输入
1 输出
2 错误
xxx > /dev/null 2 > &1 
```

### nginx 负载均衡使用
```nginx
http {
    upstream group1{
        server 192.168.1.1:80 weight=1;
        server 192.168.1.2:80 weight=1;
    }
    server{
        location /a/ {
            proxy_pass http://group1/;
        }
    }
}
```
### 新版本GIT
```bash
yum install -y epel-release  
~~rpm -ivh https://centos7.iuscommunity.org/ius-release.rpm~~
rpm -ivh https://repo.ius.io/ius-release-el7.rpm
yum list git2u  
yum install -y git2u  
git --version 
```
### npm源问题
<https://www.jianshu.com/p/010e47ed2bfd?tdsourcetag=s_pcqq_aiomsg>
```
#npm
npm install 包的名字 --registry https://registry.npm.taobao.org
npm config get registry
npm config set registry https://registry.npm.taobao.org
npm install -g nrm
nrm ls
nrm use taobao
nrm test
#yarn
yarn save 包的名字 --registry https://registry.npm.taobao.org/
yarn config get registry
yarn config set registry https://registry.npm.taobao.org/
npm install -g yrm
yrm ls
yrm use taobao
yrm test
#cgr
npm install -g cgr
```
### jenkins
```bash
docker pull jenkins/jenkins
chown 1000 /home/www/jenkins
docker run -d  -p 8090:8080 -p 50001:50000 -v `pwd`/jenkins:/var/jenkins_home --env JAVA_OPTS="-Duser.timezone=GMT+08" --privileged=true --name myjenkins jenkins/jenkins
docker logs -f myjenkins
"password like this :6******************************e"
docker inspect 4**********d

```
### rails
```bash
ruby 安装
gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
\curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
rvm install 2.6.3
gem install solargraph
gem install rails

fix:sqlite3.7
gem install sqlite3 -- --with-sqlite3-lib=/usr/local/sqlite/lib
```
### oh-my-zsh
```bash
cat /etc/shells
yum install zsh -y
chsh -s /bin/zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### Window10 包管理 scoop
* 用户名不含中文字符
* Windows 7 SP1+ / Windows Server 2008+
* PowerShell 3+
* .NET Framework 4.5+
产看版本：
```powershell
$PSVersionTable.PSVersion.Major   #查看Powershell版本
$PSVersionTable.CLRVersion.Major  #查看.NET Framework版本
```
安装scoop
```powershell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```
常用
```powershell
scoop install aria2
scoop search git
```

### Centos7 其它rpm源
rpmfusion提供了，在Fedora和 centos 源之外的其他yum 源
`Centos 7`
```bash
yum localinstall -y https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm
```
`yum install ffmpeg`

### homebrew 设置代理
```
alias proxy_ss='export all_proxy=socks5://127.0.0.1:1086'
alias proxy_v2ray='export all_proxy=socks5://127.0.0.1:1081'
alias proxy_clashx='export all_proxy=socks5://127.0.0.1:7891'
alias unproxy='unset all_proxy'
```

### 批量删除进程
```bash
ps -ef |grep 41230 | grep -v grep | awk '{print $2}'| xargs kill -9
```
- ps ： 报告当前进程的快照
- kill ： 向一个进程发出信号
- killall ： 按名字消灭进程
- pkill ： 根据名字和其它属性查看或者发出进程信号
- skill ： 发送一个信号或者报告进程状态
- xkill ： 按照X资源消灭一个客户程序

### golang 设置代理
`如果您使用的 Go 版本是 1.13 及以上 （推荐）`
```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct

# 设置不走 proxy 的私有仓库，多个用逗号相隔（可选）
go env -w GOPRIVATE=*.corp.example.com

# 设置不走 proxy 的私有组织（可选）
go env -w GOPRIVATE=example.com/org_name
```
`如果您使用的 Go 版本是 1.12 及以下`
```bash
# 启用 Go Modules 功能
export GO111MODULE=on
# 配置 GOPROXY 环境变量
export GOPROXY=https://goproxy.io
export GOPATH=/root/gopath
export GOROOT=/usr/lib/go
export GOBIN=$GOPATH/bin
```
`PowerShell (Windows)`
```bash
# 启用 Go Modules 功能
$env:GO111MODULE="on"
# 配置 GOPROXY 环境变量
$env:GOPROXY="https://goproxy.io"
```

### deno 安装
```bash
curl -fsSL https://x.deno.js.cn/install.sh | sh
curl -fsSL https://x.deno.js.cn/install.sh | sh -s v1.0.0
```

### monitor and restart
```sh
#! /bin/sh
 
host_dir=`echo ~`          # 当前用户根目录
proc_name="/home/wkubuntu/named/sbin/named"        # 进程名
file_name="/mnt/bindmonitor.log"       # 日志文件
pid=0
 
proc_num()            # 计算进程数
{
 num=`ps -ef | grep $proc_name | grep -v grep | wc -l`
 return $num
}
 
proc_id()            # 进程号
{
 pid=`ps -ef | grep $proc_name | grep -v grep | awk '{print $2}'`
}
 
proc_num
number=$?
if [ $number -eq 0 ]         # 判断进程是否存在
then
 /home/wkubuntu/named/sbin/named -c /home/wkubuntu/named/etc/named.conf -n 1 &
              # 重启进程的命令，请相应修改
 proc_id           # 获取新进程号
 echo ${pid}, `date` >> $file_name  # 将新进程号和重启时间记录
fi
```