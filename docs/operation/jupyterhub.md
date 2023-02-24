# 安装jupyterhub

npm install -g configurable-http-proxy
sudo pip3 install jupyterhub
sudo conda install notebook
sudo pip3 install jupyterlab

# 生成配置文件

jupyterhub --generate-config

# 启动

cd /home/geovis/.local/bin

nohup jupyterhub -f jupyterhub_config.py  &

# 默认8000端口，访问链接，查看安装情况

http://127.0.0.1:8000/

创建用户

useradd muchao

修改密码

passwd muchao

**修改文件夹权限**

chmod -R 777 home
cd /home
mkdir muchao
chown muchao:muchao muchao -R

安装jupyter后，使用时显示找不到命令(command not found)

发现了jupyter位置，注意bin文件是存放命令的，所以我们把它添加到环境变量

sudo vim /etc/profile
添加如下代码
export PATH=$PATH:~/.local/bin
退出编辑
source  /etc/profile





# JupyterLab

**JupyterLab 安全验证**
JupyterLab 使用密码保证服务的安全，从而确保其他用户无法登录使用。默认情况下，会自动生成随机密码，如 ： http://127.0.0.1:8888/lab?token=2cefb80900a38689d9d0d2c4927832fa4ae322b1e441c601，其中 token=密码。

为方便远程登录，可手动设置密码，方法如下：

生成配置文件
jupyter server --generate-config

该命令会在 ~/.jupyter 目录下生成配置文件 jupyter_server_config.py, 如果该配置文件已经存在，则会提示是否替换该文件。

手动设置密码
jupyter server password

在终端输入密码后，会将该密码的哈希值写入配置文件。

你现在就可以尝试打开 JupyterLab，你会发现你需要输入刚刚设定的密码才可以登录。


打开jupyter_server_config.py,文件并添加一下代码

```python
c.ServerApp.allow_origin = '*'
c.ServerApp.ip = '192.168.1.101'
```

c.ServerApp.allow_origin = ‘*’ 代表接受远程访问。 这一点大部分教程都是正确的



输入以下命令启动

```bash
sudo nohup jupyter-lab --allow-root & 
```