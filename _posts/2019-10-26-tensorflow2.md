
[TOC]


### 创建环境
```bash
python3 -m venv py3-tf-env
source py3-tf-env/bin/activate
pip install -U pip setuptools
```

### 安装依赖
```bash
pip install numpy pandas matplotlib sklearn tensorflow==2.0.0rc2 jupyter
```

### jupyter 配置
```bash

pip install --upgrade jupyterthemes
jt -t onedork -T -N

pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install

pip install qgrid
jupyter nbextension enable --py --sys-prefix widgetsnbextension
```

### 本地运行
```bash
jupyter notebook
```

### 远程配置
* 配置文件
```bash
jupyter notebook --generate-config
```
* 生成密码
```ipython
from notebook.auth import passwd
passwd()
'sha1:43b95b731276:5d330ee6f6054613b3ab4cc59c5048ff7c70f549'
```
* 修改默认配置文件
```ini
修改以下内容并去掉前面的注释 #：
c.NotebookApp.ip='*' #设置访问notebook的ip，*表示所有IP，这里设置ip为都可访问
c.NotebookApp.password=u'sha1:5df252f58b7f:bf65d53125bb36c085162b3780377f66d73972d1' #填写刚刚生成的密文
c.NotebookApp.open_browser = False # 禁止notebook启动时自动打开浏览器
c.NotebookApp.port =8889 #指定访问的端口，默认是8888。
```
* 启动
```bash
jupyter notebook --no-browser
jupyter notebook --allow-root 
```