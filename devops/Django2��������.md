系统：CentOS 7.4
MySQL：5.7
python：3.6.9

## 一、 准备基础环境：

### 1. 安装python编译依赖软件：

```
[root@lxk ~]# ~]# yum install -y git
[root@lxk ~]# ~]# yum install -y gcc make patch gdbm-devel openssl-devel sqlite-devel readline-devel zlib-devel bzip2-devel
[root@lxk ~]# ~]# useradd python
[root@lxk ~]# ~]# wget https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer
[root@lxk ~]# ~]# chmod +x pyenv-installer
[root@lxk ~]# ~]# ./pyenv-installer			#安装pyenv
```

在python用户的~/.bash_profile中追加以下内容

```
export PATH=”/home/python/.pyenv/bin:$PATH”
eval “$(pyenv init -)”
eval “$(pyenv virtualenv-init -)”
```

添加完成后运行：source ~/.bash_profile

**pyenv命令使用说明：**

```
[python@pkg ~]$ pyenv help
Usage: pyenv <command> [<args>]

Some useful pyenv commands are:
   commands    List all available pyenv commands		#列出可用命令
   local       Set or show the local application-specific Python version	#
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using python-build
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable 

[python@pkg ~]$ pyenv help install
Usage: pyenv install [-f] [-kvp] <version>
       pyenv install [-f] [-kvp] <definition-file>
       pyenv install -l|--list
       pyenv install --version

  -l/--list          List all available versions	#列出已安装的版本
  -f/--force         Install even if the version appears to be installed already		#强制安装，即使版本已安装
  -s/--skip-existing Skip if the version appears to be installed already	#跳过已安装的版本

  python-build options:

  -k/--keep          Keep source tree in $PYENV_BUILD_ROOT after installation
                     (defaults to $PYENV_ROOT/sources)
  -p/--patch         Apply a patch from stdin before building
  -v/--verbose       Verbose mode: print compilation status to stdout
  --version          Show version of python-build
  -g/--debug         Build a debug version
```

### 2. 安装python并设置虚拟环境

```
~]# pyenv install 3.6.6 -v
#运行后会连网下载3.5.3的包，但速度很慢，可在~/.pyenv/cache下放置3.5.3的压缩包，再执行安装命令
```

创建python虚拟环境

```
[python@master devops]$ pyenv virtualenv 3.6.9 py369
Looking in links: /tmp/tmpvj1izhx2
Requirement already satisfied: setuptools in /home/python/.pyenv/versions/3.6.9/envs/py369/lib/python3.6/site-packages (40.6.2)
Requirement already satisfied: pip in /home/python/.pyenv/versions/3.6.9/envs/py369/lib/python3.6/site-packages (18.1)
```

设置/home/python/devops目录运行环境为py369

```
[python@master devops]$ pyenv local py369
(py369) [python@master devops]$ 
```

### 3. pip通用配置
在~/.pip/pip.conf添加以下内容：

```
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

### 4. 安装ipython及django 2.2.0

```
(py369) [python@master devops]$ pip install django==2.2.0
(py369) [python@master devops]$ pip install ipython
(py369) [python@master devops]$ python -m django --version
2.2
```

### 5. 安装配置mysql及安装mysql开发工具、mysql连接工具

```
[root@master ~]# wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
[root@master ~]# rpm -qi mysql80-community-release-el7-3.noarch.rpm
[root@master ~]# yum install -y yum-utils
[root@master ~]# yum-config-manager --enable mysql57-community		#启用mysql5.7版本
[root@master ~]# yum-config-manager --disable mysql80-community		#禁用mysql8.0版本
[root@master ~]# yum install -y mysql-community-server mysql-community-devel
```
**配置mysql**

```
编辑/etc/my.cnf,添加以下内容：
character-set-server = utf8
collation-server = utf8_general_ci
[client]
default-character-set = utf8
```
**mysql创建用户、库**

```
[root@master ~]# mysql -uroot -hlocalhost -p
mysql> create database devops CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on *.* to 'devops'@'%' IDENTIFIED BY 'Devops@1234';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```
**安装python3的mysql扩展**

```
(py369) [python@master devops]$ pip install mysqlclient
#出现以下报错，检查mysql-devel是否安装。
Looking in indexes: https://mirrors.aliyun.com/pypi/simple/
Collecting mysqlclient
  Downloading https://mirrors.aliyun.com/pypi/packages/d0/97/7326248ac8d5049968bf4ec708a5d3d4806e412a42e74160d7f266a3e03a/mysqlclient-1.4.6.tar.gz (85 kB)
     |████████████████████████████████| 85 kB 5.8 MB/s 
    ERROR: Command errored out with exit status 1:
     command: /home/python/.pyenv/versions/3.6.9/envs/py369/bin/python3.6 -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-j4nqaqbh/mysqlclient/setup.py'"'"'; __file__='"'"'/tmp/pip-install-j4nqaqbh/mysqlclient/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-install-j4nqaqbh/mysqlclient/pip-egg-info
         cwd: /tmp/pip-install-j4nqaqbh/mysqlclient/
    Complete output (12 lines):
    /bin/sh: mysql_config: command not found
    /bin/sh: mariadb_config: command not found
    /bin/sh: mysql_config: command not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-install-j4nqaqbh/mysqlclient/setup.py", line 16, in <module>
        metadata, options = get_config()
      File "/tmp/pip-install-j4nqaqbh/mysqlclient/setup_posix.py", line 61, in get_config
        libs = mysql_config("libs")
      File "/tmp/pip-install-j4nqaqbh/mysqlclient/setup_posix.py", line 29, in mysql_config
        raise EnvironmentError("%s not found" % (_mysql_config_path,))
    OSError: mysql_config not found			
    ----------------------------------------
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
```

**测试mysql是否可正常连接**

```
(py369) [python@master devops]$ python
Python 3.6.9 (default, Mar 22 2020, 18:40:11) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> import MySQLdb
>>> conn = MySQLdb.connect(host='localhost',port = 3306, user='devops', passwd='Devops@1234', db='devops',)
>>> cur = conn.cursor()
```

## 二、 第一个Django Project

### 1. 创建一个Project

```
(py369) [python@master project]$ django-admin startproject devops
(py369) [python@master project]$ tree devops/
devops/
├── devops
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 5 files
(py369) [python@master project]$ 
```
### 2. 运行项目
修改配置文件settings.py，配置允许访问地址、数据库连接信息、语言信息

```
ALLOWED_HOSTS = ['*']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'devops',
        'HOST': '127.0.0.1',
        'USER': 'devops',
        'PASSWORD': 'Devops@1234',
        'PORT': 3306,
    }
}

LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
```
启动服务后，访问服务器出现django页面即为正常（需在project/devops目录下运行，运行目录有manage.py文件）：
```
(py369) [python@master devops]$ python manage.py runserver 0.0.0.0:8080
```

### 3. 让django第一个项目admin跑起来

上一步运行项目时提示如下：
```
You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```
按照提示执行把SQL导入数据库：
```
(py369) [python@master devops]$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK
```

创建管理员用户：
```
(py369) [python@master devops]$ python manage.py createsuperuser
用户名 (leave blank to use 'python'): admin			#默认为python用户
电子邮件地址: lixinkuan@163.com
Password: 
Password (again): 
Superuser created successfully.
```

再次运行项目即可访问/admin页面、登陆管理员、创建用户、给用户授权



### 4. 创建一个名为hello的应用

```
(py369) [python@master devops]$ python manage.py startapp hello
(py369) [python@master devops]$ vim devops/settings.py 
#在INSTALLED_APPS中添加'hello.apps.HelloConfig',
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'hello.apps.HelloConfig',
]
(py369) [python@master devops]$ vim devops/urls.py 	#在主路由中添加跳转至hello的子路由

from hello import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', views.index),
]

from django.urls import path

(py369) [python@master devops]$ vim hello/urls.py 	#编写子路由规则
from . import views
urlpatterns = [
    path('hello/', views.index, name='index')
]

(py369) [python@master devops]$ vim hello/views.py 	#编写页面内容
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.

def index(request):
    return HttpResponse("<p>Hello World, Hello, Django</p>")

```
通过以上方法可使用IP:PORT/hello/hello来访问资源。