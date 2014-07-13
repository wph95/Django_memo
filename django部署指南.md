# Django 部署指南
##           by wph95


------
当我们辛勤苦苦的码完网站主体之后，就需要用合理的方法挂到服务器上去
以下就讲展示基于  
> * [Django](https://www.djangoproject.com/)
> * [Gunicorn](https://gunicorn.org/)
> * [Nginx](http://nginx.org/)

的部署方法

------

## 前期准备
首先 你要用virtualenv用于创建独立的Python环境 使多个python多个Python相互独立，互不影响
你比较高大上 用virtualenvwrapper   也没关系。。。

```bash
virtualenv --no-site-packages test
  cd test
  source bin/activate
  pip install gunicorn django
  django-admin.py startproject hello
  cd hello
  # to test the base setup works
  python manage.py runserver 0.0.0.0:8000

```
------

## Gunicorn
Gunicorn“绿色独角兽”是一个被广泛使用的高性能的Python WSGI UNIX HTTP服务器
 与django搭配 效果拔群
Testing Django with Gunicorn is as simple as: 
我们先来个simple Example来运行django
```bash
gunicorn_django -b 0.0.0.0:8000
```

当然 正常的合理的运行方法是写个sh脚本来运行 like it
```dash
 #!/bin/bash
  set -e
  LOGFILE=/var/log/gunicorn/hello.log
  LOGDIR=$(dirname $LOGFILE)
  NUM_WORKERS=3
  # user/group to run as
  USER=your_unix_user
  GROUP=your_unix_group
  cd /path/to/test/hello
  source ../bin/activate
  test -d $LOGDIR || mkdir -p $LOGDIR
  exec ../bin/gunicorn_django -w $NUM_WORKERS 
    --user=$USER --group=$GROUP --log-level=debug 
    --log-file=$LOGFILE 2>>$LOGFILE
```
看起来很麻烦？ 这里在举一个实际中使用的脚本供参考
```bash
#!/bin/bash
set -e
LOGFILE=/var/log/blog.log
# log路径
LOGDIR=$(dirname $LOGFILE)
NUM_WORKERS=1
#下面有解释
# user/group to run as
USER=root
GROUP=root
cd /home/www/blog/
#web的路径
test -d $LOGDIR || mkdir -p $LOGDIR
exec gunicorn -w $NUM_WORKERS -b 0.0.0.0:8000 lintcode.wsgi:application --user=$USER --group=$GROUP
#8000是端口 可随意修改
```
ps 
1.这里要解释下NUM_WORKERS参数的设置 
gunicorn只需要启用4–12个workers，就足以每秒钟处理几百甚至上千个请求了。
如果你的网站访问量较大，我推荐的合理的worker数量是：(2 x $num_cores) 
2.可能你要提升下脚本权限 like
```bash
chmod ug+x script.sh
```
------

## 守护进程
这里有两种选择来维护上面的脚本持续运行
一种是 利用 Supervisor

另一种是 Upstart
### Upstart
Upstart是替换 /sbin/init 守护进程
这是一个例子 新建/etc/init/hello.conf 
```bash
  description "Test Django instance"
  start on runlevel [2345]
  stop on runlevel [06]
  respawn
  respawn limit 10 5
  exec /test/hello/script.sh
  #脚本路径为/test/hello/script.sh
```
运行下面指令 来试试吧
```
service hello {start,status,stop} 
#(as root).
```
ps.如果上面指令没效果 可能是链接没做好 执行下面的指令
```
  sudo ln -s /lib/init/upstart-job /etc/init.d/hello
```
么么哒 django就这样安全的部署好喽 
