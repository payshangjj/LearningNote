> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.jianshu.com/p/434cc22c6240

用 Python 如何写一个接口呢，首先得要有数据，可以用我们在网站上爬的数据，在上一篇文章中写了如何用 Python 爬虫，有兴趣的可以看看：

[https://www.cnblogs.com/sixrain/p/9120529.html](https://www.cnblogs.com/sixrain/p/9120529.html)

大量的数据保存到数据库比较方便。我用的 pymsql，pymsql 是 Python 中操作 MySQL 的模块，其使用方法和 MySQLdb 几乎相同。但目前在 python3.x 中，PyMySQL 取代了 MySQLdb。

**1、连接数据库**

<pre># 连接数据库，需指定charset否则可能会报错
db = pymysql.connect(host="localhost", user="root", password="123", db="mysql", charset="utf8mb4")
cursor = db.cursor()  # 创建一个游标对象

</pre>

**2、创建数据库**

<pre>cursor.execute("DROP TABLE IF EXISTS meizi_meizis")  # 如果表存在则删除
     # 创建表sql语句
     createTab = """create table meizi_meizis(
             id int primary key auto_increment,
             mid varchar(10) not null,
             title varchar(50),
             picname varchar(10),
             page_url varchar(50),
             img_url varchar(50)
             );"""
     cursor.execute(createTab)  # 执行创建数据表操作

</pre>

**3、爬取数据**

<pre>def html(self, href, title):
      lists = []
      meiziid = href.split('/')[-1]
      html = self.request(href)
      max_span = BeautifulSoup(html.text, 'lxml').find('div', class_='pagenavi').find_all('span')[-2].get_text()
      for page in range(1, int(max_span) + 1):
          meizi = {}
          page_url = href + '/' + str(page)
          img_html = self.request(page_url)
          img_url = BeautifulSoup(img_html.text, 'lxml').find('div', class_='main-image').find('img')['src']
          picname = img_url[-9:-4]
          meizi['meiziid'] = meiziid
          meizi['title'] = title
          meizi['picname'] = picname
          meizi['page_url'] = page_url
          meizi['img_url'] = img_url
          lists.append(meizi)  # 保存到返回数组中
      return lists

</pre>

**4、保存到数据库**

<pre>def all_url(self, url):
      html = self.request(url)
      all_a = BeautifulSoup(html.text, 'lxml').find('div', class_='all').find_all('a')
      for index, a in enumerate(all_a):
          title = a.get_text()
          href = a['href']
          lists = self.html(href, title)
          for i in lists:
              # print(i['meiziid'], i['title'], i['picname'], i['page_url'], i['img_url'])
              # 插入数据到数据库sql语句，%s用作字符串占位
              sql = "INSERT INTO `meizi_meizis`(`mid`,`title`,`picname`,`page_url`,`img_url`) VALUES(%s,%s,%s,%s,%s)"
              try:
                  cursor.execute(sql, (i['meiziid'], i['title'], i['picname'], i['page_url'], i['img_url']))
                  db.commit()
                  print(i[0] + " is success")
              except:
                  db.rollback()
      db.close()  # 关闭数据库

</pre>

**5、创建 web 工程**

运行我们的爬虫，很快数据库表里就有数据了。

![](http://upload-images.jianshu.io/upload_images/7209287-b54bcd2de7e3fb27.png)

**然后开始写接口。我是通过 Django+rest_framework 来写的。Django 是用 Python 开发的一个免费开源的 Web 框架，可以用于快速搭建高性能，优雅的网站**。Django 中提供了开发网站经常用到的模块，常见的代码都为你写好了，减少重复的代码。

**Django 目录结构**

**urls.py** （[https://code.ziqiangxuetang.com/django/django-views-urls.html](https://code.ziqiangxuetang.com/django/django-views-urls.html)）—— 网址入口，关联到对应的 [views.py](http://views.py) 中的一个函数（或者 generic 类），访问网址就对应一个函数。

**views.py** （[https://code.ziqiangxuetang.com/django/django-views-urls.html](https://code.ziqiangxuetang.com/django/django-views-urls.html)）—— 处理用户发出的请求，从 [urls.py](http://urls.py) 中对应过来, 通过渲染 templates 中的网页可以将显示内容，比如登陆后的用户名，用户请求的数据，输出到网页。

**models.py** （[https://code.ziqiangxuetang.com/django/django-models.html](https://code.ziqiangxuetang.com/django/django-models.html)）—— 与数据库操作相关，存入或读取数据时用到这个，当然用不到数据库的时候 你可以不使用。

**forms.py** （[https://code.ziqiangxuetang.com/django/django-forms.html](https://code.ziqiangxuetang.com/django/django-forms.html)）—— 表单，用户在浏览器上输入数据提交，对数据的验证工作以及输入框的生成等工作，当然你也可以不使用。

templates 文件夹

**views.py** —— 中的函数渲染 templates 中的 Html 模板，得到动态内容的网页，当然可以用缓存来提高速度。

**admin.py** （[https://code.ziqiangxuetang.com/django/django-admin.html](https://code.ziqiangxuetang.com/django/django-admin.html)）—— 后台，可以用很少量的代码就拥有一个强大的后台。

**settings.py** （[https://code.ziqiangxuetang.com/django/django-settings.html](https://code.ziqiangxuetang.com/django/django-settings.html)）—— Django 的设置，配置文件，比如 DEBUG 的开关，静态文件的位置等。

**Django 常用操作**

1）新建一个 django project

django-admin.py startproject project_name

2）新建 app

python manage.py startapp app_name

一般一个项目有多个 app, 当然通用的 app 也可以在多个项目中使用。

还得在工程目录的 settings.py 文件在配置

<pre>INSTALLED_APPS = [
   'django.contrib.admin',
   'django.contrib.auth',
   'django.contrib.contenttypes',
   'django.contrib.sessions',
   'django.contrib.messages',
   'django.contrib.staticfiles',
   'rest_framework',

   'meizi',
]

</pre>

在 app/views.py 下编写代码

<pre>def index(request):
   return HttpResponse(u"你好")

</pre>

在工程目录 urls.py 配置

<pre>from learn import views as learn_views
urlpatterns = [
   url(r'^/pre>, learn_views.index), 
]

</pre>

通过 python manage.py runserver 启动，就会看到我们输出的 “你好” 了

3）创建数据库表 或 更改数据库表或字段

在 app 下的 models.py 创建表

<pre>class Person(models.Model):
   name = models.CharField(max_length=30)
   age = models.IntegerField()

   def __unicode__(self):
       # 在Python3中使用 def __str__(self):
       return self.name

</pre>

运行命令，就可以生成对应的表

<pre>Django 1.7.1及以上 用以下命令
# 1\. 创建更改的文件
python manage.py makemigrations
# 2\. 将生成的py文件应用到数据库
python manage.py migrate

</pre>

在 views.py 文件里就可以获取数据库的数据

<pre>def create(request):
   # 新建一个对象的方法有以下几种：
   Person.objects.create(name='xiaoli', age=18)
   # p = Person(, age=23)
   # p = Person()
   # p.age = 23
   # p.save()
   # 这种方法是防止重复很好的方法，但是速度要相对慢些，返回一个元组，第一个为Person对象，
   # 第二个为True或False, 新建时返回的是True, 已经存在时返回False
   # Person.objects.get_or_create(, age=23)
   s = Person.objects.get(name='xiaoli')
   return HttpResponse(str(s))

</pre>

**6、写接口**

接口使用 rest_framework，rest_framework 是一套基于 Django 的 REST 框架，是一个强大灵活的构建 Web API 的工具包。

**写接口三步完成：连接数据库、取数据、数据输出**

##### **1）连接数据库**

在工程目录下的 settings.py 文件下配置

<pre>DATABASES = {
   # 'default': {
   #     'ENGINE': 'django.db.backends.sqlite3',
   #     'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
   # }
   'default': {
       'ENGINE': 'django.db.backends.mysql',
       'NAME': 'mysql',
       'USER': 'root',
       'HOST': '127.0.0.1',
       'PASSWORD': '123',
       'PORT': 3306,
       # show variables like 'character_set_database'；
       # 修改字段字符编码
       # alter table spiders_weibo modify text longtext charset utf8mb4 collate utf8mb4_unicode_ci;
       'OPTIONS': {'charset': 'utf8mb4'},
   }
}

</pre>

**2）取数据**

既然要取数据，那 model 肯定得和数据库的一致，我发现一个快捷的方式可以把数据库中的表生成对应的 model，在项目目录下执行命令

<pre>python manage.py inspectdb

</pre>

可以看到下图

![](http://upload-images.jianshu.io/upload_images/7209287-274ea4b3c0a79a28)

取我们表的 model 拷贝到 app 下的 models.py 里

<pre>class Meizis(models.Model):
   mid = models.CharField(max_length=10)
   title = models.CharField(max_length=50, blank=True, null=True)
   picname = models.CharField(max_length=10, blank=True, null=True)
   page_url = models.CharField(max_length=50, blank=True, null=True)
   img_url = models.CharField(max_length=50, blank=True, null=True)

   class Meta:
       managed = False
       db_table = 'meizi_meizis'

</pre>

**创建一个序列化 Serializer 类**

提供序列化和反序列化的途径，使之可以转化为，某种表现形式如 json。我们可以借助 serializer 来实现，类似于 Django 表单（form）的运作方式。在 app 目录下，创建文件 serializers.py。

<pre>class MeiziSerializer(serializers.ModelSerializer):
   # ModelSerializer和Django中ModelForm功能相似
   # Serializer和Django中Form功能相似
   class Meta:
       model = Meizis
       # 和"__all__"等价
       fields = ('mid', 'title', 'picname', 'page_url', 'img_url')

</pre>

这样在 views.py 就可以来获取数据库的数据了

<pre>meizis = Meizis.objects.all()
serializer = MeiziSerializer(meizis, many=True)
return Response(serializer.data)

</pre>

**3) 数据输出客户端或前端**

REST 框架提供了两种编写 API 视图的封装。

*   @api_view 装饰器，基于方法的视图。

*   继承 APIView 类，基于类的视图。

request.data 会自行处理输入的 json 请求
使用格式后缀明确的指向指定的格式，需要添加一个 format 关键字参数
http [http://127.0.0.1:8000/getlist.json](http://127.0.0.1:8000/getlist.json) # JSON 后缀
[http://127.0.0.1:8000/getlist.api](http://127.0.0.1:8000/getlist.api) # 可视化 API 后缀
[http://127.0.0.1:8000/getlist/](http://127.0.0.1:8000/getlist/) code="print 123"post

<pre>@api_view(['GET', 'POST'])
def getlist(request, format=None):
   if request.method == 'GET':
       meizis = Meizis.objects.all()
       serializer = MeiziSerializer(meizis, many=True)
       return Response(serializer.data)

   elif request.method == 'POST':
       serializer = MeiziSerializer(data=request.data)
       if serializer.is_valid():
           serializer.save()
           return Response(serializer.data, status=status.HTTP_201_CREATED)
       return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

</pre>

**4）分页**

最后别忘了在 urls.py 配置 URL，通过浏览器就可以看到 json 数据了。当然 app 也是可以调用我们的接口的。

##### **还有个问题**

我们的数据有好几千条，一块返回来很不合理，所以需要分页，当然 rest_framework 框架提供了这个功能，post 请求不支持，需要自己查数据库或者切片来进行返回。**来看看 rest_framework 是如何来分页的**。在 models.py 里创建一个类

<pre>class StandardResultSetPagination(LimitOffsetPagination):
   # 默认每页显示的条数
   default_limit = 20
   # url 中传入的显示数据条数的参数
   limit_query_param = 'limit'
   # url中传入的数据位置的参数
   offset_query_param = 'offset'
   # 最大每页显示条数
   max_limit = None

</pre>

在 serializers.py 创建俩个类，为什么是俩个？因为我们有俩个接口，一个明细，一个列表，而列表是不需要把字段的所有数据都返回的

<pre>class ListSerialize(serializers.ModelSerializer):
   class Meta:
       model = Meizis
       fields = ('mid', 'title')

class ListPicSerialize(serializers.ModelSerializer):
   class Meta:
       model = Meizis
       fields = "__all__"

</pre>

在 views.py 里编写

<pre>@api_view(['GET'])
def getlist(request, format=None):
   if request.method == 'GET':
       meizis = Meizis.objects.values('mid','title').distinct()
       # http: // 127.0.0.1:8000 / getlist?limit = 20
       # http: // 127.0.0.1:8000 / getlist?limit = 20 & offset = 20
       # http: // 127.0.0.1:8000 / getlist?limit = 20 & offset = 40
       # 根据url参数 获取分页数据
       obj = StandardResultSetPagination()
       page_list = obj.paginate_queryset(meizis, request)
       # 对数据序列化 普通序列化 显示的只是数据
       ser = ListSerialize(instance=page_list, many=True)  # 多个many=True # instance：把对象序列化
       response = obj.get_paginated_response(ser.data)
       return response

@api_view(['GET', 'POST'])
def getlispic(request, format=None):
   if request.method == 'GET':
       mid = request.GET['mid']
       if mid is not None:
           # get是用来获取一个对象的，如果需要获取满足条件的一些数据，就要用到filter
           meizis = Meizis.objects.filter(mid=mid)
           obj = StandardResultSetPagination()
           page_list = obj.paginate_queryset(meizis, request)
           ser = ListPicSerialize(instance=page_list, many=True)
           response = obj.get_paginated_response(ser.data)
           return response
       else:
           return Response(str('请传mid'))

</pre>

到这里就完成了接口的编写，都是对框架的简单使用，希望对大家有帮助。

GitHub 地址，欢迎 star

爬虫项目：[https://github.com/peiniwan/Spider2](https://github.com/peiniwan/Spider2)
Web 项目：[https://github.com/peiniwan/mysite](https://github.com/peiniwan/mysite)
APP 项目：[https://github.com/peiniwan/Ganhuo](https://github.com/peiniwan/Ganhuo)

> 作者：六月的雨
> 转载 | 原文链接：[https://www.cnblogs.com/sixrain/p/9138442.html](https://www.cnblogs.com/sixrain/p/9138442.html)
> （如有侵权，请联系删除）

#### 最新公告通知

第 19 期【Python 自动化运维入门】正在火热招生中
第 8 期 【Python 自动化运维进阶】正在火热招生中

![](http://upload-images.jianshu.io/upload_images/7209287-fafaf372fd5bc46a.png)