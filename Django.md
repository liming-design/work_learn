```
pip install django

django-admin.py startproject mysite

cd mysite
python manage.py startapp blog

在settings.py文件中将上面创建的APP也就是blog添加进来：

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
]

```

现在我们打开blog目录下的models.py文件，在文件里新建的类就是一张表。打开mysite/blog/models.py 文件进行添加类（也就是添加表)：
```
from django.db import models
class UserInfo(models.Model):
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=32)
    age = models.IntegerField()

```

5：admin后台注册表 修改app的admin.py文件：把models创建的表添加到admin后台中。
```
from django.contrib import admin

  

# Register your models here.

from questions import models

admin.site.register(models.Question)
```

```

python manage.py makemigrations questions
python manage.py migrate

python manage.py createsuperuser


python manage.py runserver


(
	function(){ }
)(jQuery);
```
### 删除数据库
```
rm -rf questions/__pycache__/
rm -rf questions/migrations/
rm -rf db.sqlite3
python3 manage.py makemigrations questions
python3 manage.py migrate
```



## 允许内嵌界面

在settings.py中
```python
X_FRAME_OPTIONS = 'SAMEORIGIN'
```

## 升级sqlite3
```
yum install -y wget

tar zxvf sqlite-autoconf-3290000.tar.gz

cd sqlite-autoconf-3290000/
./configure --prefix=/usr/local
make && make install
```
### 替换系统sqlite3版本
```
mv /usr/bin/sqlite3  /usr/bin/sqlite3_old
ln -s /usr/local/bin/sqlite3   /usr/bin/sqlite3
```
配置系统lib库
```
echo "/usr/local/lib" > /etc/ld.so.conf.d/sqlite3.conf
ldconfig
```