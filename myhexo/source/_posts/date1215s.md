---
title: 常记Fifty
date: 2019-02-01 13:14:21
tags: [python,docker]
---

### 1 linux查看端口命令

```python
netstat -ntlp   			# 查看当前所有tcp端口

netstat -ntulp |grep 80   	# 查看所有80端口使用情况

netstat -an | grep 3306   	# 查看所有3306端口使用情况

kill -9 3306				# 杀死使用3306端口的进程
```

### 2 ModelViewSet 路由

```python
from django.urls import path
from rest_framework.routers import DefaultRouter

from user import views

router = DefaultRouter()
router.register(r'user', views.UserViewSet)

urlpatterns += router.urls
```

### 3 Vue模块安装错误

#### 3.1 出现问题

```vue
94% asset optimization
ERROR  Failed to compile with 2 errors                                                                         19:47:42

These dependencies were not found:

* babel-runtime/core-js/json/stringify in ./src/http/index.js
* babel-runtime/core-js/promise in ./src/http/index.js

To install them, you can run: npm install --save babel-runtime/core-js/json/stringify babel-runtime/core-js/promise
```

#### 3.2 解决方案

```vue
// 提示
// 分别安装几个包，就可以解决了！
>cnpm install --save babel-runtime
>cnpm install --save core-js
>cnpm install --save json
>cnpm install --save stringify
```

### 4 Serializer修改数据

#### 4.1 models.py

```python
class Books(BaseModel):
    name = models.CharField(max_length=30)
    publish = models.CharField(max_length=30)
    read_num = models.IntegerField()  # 阅读量
    comment_num = models.IntegerField()  # 评论量

    class Meta:
        db_table = 'tb_books'

    def __str__(self):
        return self.name
```

#### 4.2 serializer.py

```python
class BooksModelSer(serializers.ModelSerializer):
    class Meta:
        model = Books
        fields = '__all__'
```

#### 4.3 views.py

```python
class BooksView(APIView):
    def put(self,request):
        print(request.data)
        id = request.data.get('id')
        books_obj = Books.objects.get(id=id)
        books_ser = BooksModelSer(data=request.data,instance=books_obj)
        books_ser.is_valid()
        books_ser.save()
        return Response({'msg': 'ok', 'code': 200})
```

#### 4.4 urls.py

```python
path('books/',BooksView.as_view())
```

### 5 element-ui 安装命令

```python
cnpm i element-ui -S
```

### 6 json获取request的数据

```python
body_json = request.body.decode()
body_dict = json.loads(body_json)
```

### 7 跨域后端

```python
INSTALLED_APPS = [
	'corsheaders'
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware'
]

CORS_ORIGIN_WHITELIST = (
    'http://127.0.0.1:8080',
    'http://localhost:8080',
)
CORS_ALLOW_CREDENTIALS = True
```

### 8 跨域前端

#### 8.1 http.js

```js
import axios from 'axios'

// 第一步：设置axios
axios.defaults.baseURL = "http://192.168.56.100:1594/"

//全局设置网络超时
axios.defaults.timeout = 10000;

//设置请求头信息
axios.defaults.headers.post['Content-Type'] = 'application/json';
axios.defaults.headers.put['Content-Type'] = 'application/json';


// 第二：设置拦截器
/**
 * 请求拦截器(当前端发送请求给后端前进行拦截)
 * 例1：请求拦截器获取token设置到axios请求头中，所有请求接口都具有这个功能
 * 例2：到用户访问某一个页面，但是用户没有登录，前端页面自动跳转 /login/ 页面
 */
axios.interceptors.request.use(
    config => {
        // 每次发送请求之前判断是否存在token，如果存在，则统一在http请求的header都加上token，不用每次请求都手动添加了
        const token = localStorage.getItem("token")
            // console.log(token)
        if (token) {
            config.headers.Authorization = 'JWT ' + token
        }
        return config;
    },
    error => {
        return Promise.error(error);
    })

axios.interceptors.response.use(
    // 请求成功
    res => res.status === 200 ? Promise.resolve(res) : Promise.reject(res),
    // 请求失败
    error => {
        if (error.response) {
            // 判断一下返回结果的status == 401？  ==401跳转登录页面。  ！=401passs
            // console.log(error.response)
            if (error.response.status === 401) {
                // 跳转不可以使用this.$router.push方法、
                // this.$router.push({path:'/login'})
                window.location.href = "http://127.0.0.1:8888/"
            } else {
                // errorHandle(response.status, response.data.message);
                return Promise.reject(error.response);
            }
            // 请求已发出，但是不在2xx的范围
        } else {
            // 处理断网的情况
            // eg:请求超时或断网时，更新state的network状态
            // network状态在app.vue中控制着一个全局的断网提示组件的显示隐藏
            // 关于断网组件中的刷新重新获取数据，会在断网组件中说明
            // store.commit('changeNetwork', false);
            return Promise.reject(error.response);
        }
    });


// 第三：封装axios请求
// 3.1 封装get请求
export function axios_get(url, params) {
    return new Promise(
        (resolve, reject) => {
            axios.get(url, {params:params})
                .then(res => {
                    // console.log("封装信息的的res", res)
                    resolve(res.data)
                }).catch(err => {
                    reject(err.data)
                })
        }
    )
}

// 3.2 封装post请求
export function axios_post(url, data) {
    return new Promise(
        (resolve, reject) => {
            // console.log(data)
            axios.post(url, JSON.stringify(data))
                .then(res => {
                    // console.log("封装信息的的res", res)
                    resolve(res.data)
                }).catch(err => {
                    reject(err.data)
                })
        }
    )
}

// 3.3 封装put请求
export function axios_put(url, data) {
    return new Promise(
        (resolve, reject) => {
            // console.log(data)
            axios.put(url, JSON.stringify(data))
                .then(res => {
                    // console.log("封装信息的的res", res)
                    resolve(res.data)
                }).catch(err => {
                    reject(err.data)
                })
        }
    )
}

// 3.4 封装delete请求
export function axios_delete(url, data) {
    return new Promise(
        (resolve, reject) => {
            // console.log(data)
            axios.delete(url, { params: data })
                .then(res => {
                    // console.log("封装信息的的res", res)
                    resolve(res.data)
                }).catch(err => {
                    reject(err.data)
                })
        }
    )
}

```

#### 8.2 apis.js

```js
//将我们http.js中封装好的  get,post.put,delete  导过来
import { axios_get, axios_post, axios_delete, axios_put } from './index.js'


// 书籍管理接口
export const getBookList = (params, headers) => axios_get("/books/book/", params, headers)
export const addBook = (params, headers) => axios_post("/books/book/", params, headers)
export const updateBook = (params, headers) => axios_put("/books/book/", params, headers)
export const delBook = (params, headers) => axios_delete("/books/book/", params, headers)


//按照格式确定方法名
export const user_login = p => axios_post("/user/login/", p)  //
export const get_dept_list = p => axios_get("/account/deptManage/", p)  //
```

### 9 继承AbstractUser

```python
AUTH_USER_MODEL = 'user.User'

from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    username = models.CharField(max_length=30, unique=True)
```

### 10 settings 配置

```python
"""
Django settings for opwf project.

Generated by 'django-admin startproject' using Django 2.0.13.

For more information on this file, see
https://docs.djangoproject.com/en/2.0/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/2.0/ref/settings/
"""
import datetime
import os, sys

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.0/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'uorj1ni^mnut@wo@c%)iv)%5=8dxlml4-j0!f3b%4#f*8a5)3t'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ['*']


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'user.apps.UserConfig',
    'workflow.apps.WorkflowConfig',
    'workerorder.apps.WorkerorderConfig',
    # 'jwt',
    # 'rest_framework_jwt',
    # 'rest_framework.authentication'

]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'opwf.urls'
CORS_ORIGIN_ALLOW_ALL = True

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'opwf.wsgi.application'


# Database
# https://docs.djangoproject.com/en/2.0/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'opwf_db',
        'USER': 'root',
        'PASSWORD': '1',
        'HOST': '127.0.0.1',
        'PORT': '3306'
    }
}


# Password validation
# https://docs.djangoproject.com/en/2.0/ref/settings/#auth-password-validators

REST_FRAMEWORK = {
    # 文档报错： AttributeError: ‘AutoSchema’ object has no attribute ‘get_link’
    # 用下面的设置可以解决
    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.AutoSchema',
    # 默认设置是:
    # 'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.openapi.AutoSchema',

    # 异常处理器
    # 'EXCEPTION_HANDLER': 'user.utils.exception_handler',

    # Base API policies    　　默认渲染器类
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
    # 默认解析器类
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.FormParser',
        'rest_framework.parsers.MultiPartParser'
    ],
    # 1.认证器（全局）
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # 在 DRF中配置JWT认证
        # 'rest_framework.authentication.SessionAuthentication',  # 使用session时的认证器
        # 'rest_framework.authentication.BasicAuthentication'  # 提交表单时的认证器
    ],

    # 2.权限配置（全局）： 顺序靠上的严格
    'DEFAULT_PERMISSION_CLASSES': [
        # 'rest_framework.permissions.IsAdminUser',  # 管理员可以访问
        # 'rest_framework.permissions.IsAuthenticated',  # 认证用户可以访问
        # 'rest_framework.permissions.IsAuthenticatedOrReadOnly',  # 认证用户可以访问, 否则只能读取
        'rest_framework.permissions.AllowAny',  # 所有用户都可以访问
    ],
    # 3.限流（防爬虫）
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    # 3.1限流策略
    # 'DEFAULT_THROTTLE_RATES': {
    #     'user': '100/hour',  # 认证用户每小时100次
    #     'anon': '300/day',  # 未认证用户每天能访问3次
    # },

    'DEFAULT_CONTENT_NEGOTIATION_CLASS': 'rest_framework.negotiation.DefaultContentNegotiation',
    'DEFAULT_METADATA_CLASS': 'rest_framework.metadata.SimpleMetadata',
    'DEFAULT_VERSIONING_CLASS': None,

    # 4.分页（全局）：全局分页器, 例如 省市区的数据自定义分页器, 不需要分页
    # 'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    # 每页返回数量
    # 'PAGE_SIZE': 1
    # 5.过滤器后端
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        # 'django_filters.rest_framework.backends.DjangoFilterBackend', 包路径有变化
    ],

    # 5.1过滤排序（全局）：Filtering 过滤排序
    'SEARCH_PARAM': 'search',
    'ORDERING_PARAM': 'ordering',

    'NUM_PROXIES': None,

    # 6.版本控制：Versioning  接口版本控制
    'DEFAULT_VERSION': None,
    'ALLOWED_VERSIONS': None,
    'VERSION_PARAM': 'version',

    # Authentication  认证
    # 未认证用户使用的用户类型
    'UNAUTHENTICATED_USER': 'django.contrib.auth.models.AnonymousUser',
    # 未认证用户使用的Token值
    'UNAUTHENTICATED_TOKEN': None,

    # View configuration
    'VIEW_NAME_FUNCTION': 'rest_framework.views.get_view_name',
    'VIEW_DESCRIPTION_FUNCTION': 'rest_framework.views.get_view_description',

    'NON_FIELD_ERRORS_KEY': 'non_field_errors',

    # Testing
    'TEST_REQUEST_RENDERER_CLASSES': [
        'rest_framework.renderers.MultiPartRenderer',
        'rest_framework.renderers.JSONRenderer'
    ],
    'TEST_REQUEST_DEFAULT_FORMAT': 'multipart',

    # Hyperlink settings
    'URL_FORMAT_OVERRIDE': 'format',
    'FORMAT_SUFFIX_KWARG': 'format',
    'URL_FIELD_NAME': 'url',

    # Encoding
    'UNICODE_JSON': True,
    'COMPACT_JSON': True,
    'STRICT_JSON': True,
    'COERCE_DECIMAL_TO_STRING': True,
    'UPLOADED_FILES_USE_URL': True,

    # Browseable API
    'HTML_SELECT_CUTOFF': 1000,
    'HTML_SELECT_CUTOFF_TEXT': "More than {count} items...",

    # Schemas
    'SCHEMA_COERCE_PATH_PK': True,
    'SCHEMA_COERCE_METHOD_NAMES': {
        'retrieve': 'read',
        'destroy': 'delete'
    },

    # 'Access-Control-Allow-Origin':'http://localhost:8080',
    # 'Access-Control-Allow-Credentials': True

}

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/2.0/topics/i18n/

LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = False


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.0/howto/static-files/

STATIC_URL = '/static/'
AUTH_USER_MODEL = 'user.User'

# jwt载荷中的有效期设置
JWT_AUTH = {
    # 1.token前缀：headers中 Authorization 值的前缀
    'JWT_AUTH_HEADER_PREFIX': 'JWT',
    # 2.token有效期：一天有效
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
    # 3.刷新token：允许使用旧的token换新token
    'JWT_ALLOW_REFRESH': True,
    # 4.token有效期：token在24小时内过期, 可续期token
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(hours=24),
    # 5.自定义JWT载荷信息：自定义返回格式，需要手工创建
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'user.utils.jwt_response_payload_handler',
}
```

### 11 几个容易混淆的导包

```python
# ModelViewSet
from rest_framework import viewsets

# APIView
from rest_framework.views import APIView

# Serializer
from rest_framework import serializers

# DefaultRouter
from rest_framework.routers import DefaultRouter

# AbstractUser
from django.contrib.auth.models import AbstractUser
```

### 12 ModelViewSet 路由三部曲

```python
from django.urls import path
from rest_framework.routers import DefaultRouter

from user import views

router = DefaultRouter()
router.register(r'user', views.UserViewSet)

urlpatterns += router.urls
```

### 13 路由转换器

* 我们在定义一个标准路由的时候，可以给路由定义一些可变的参数，这些参数会传递给我们的视图函数

```python
from userapp.views import findgoods,findlove

urlpatterns = [
    path('goods/<int:id>/', findgoods),
    path('goods/<str:name>/', findlove)
]

注意：
路由转换器：
我们可以在参数部分写一个转换器，参数值在传递到视图函数之前，对参数进行格式转换，默认我们的参数值都是字符串str格式，如果我们想把参数值转成整型，我们可以使用int



from django.shortcuts import render, HttpResponse

# Create your views here.

goods_list=['苹果手机','联想电脑','大西瓜']

def findgoods(request, id):
    print(id, type(id))
    name = goods_list[id]
    return  HttpResponse("查找商品："+name)
```

### 14 状态码大全

```python
* http状态返回代码 1xx（临时响应）
    表示临时响应并需要请求者继续执行操作的状态代码。
代码说明：
100   （继续）请求者应当继续提出请求。服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。
101   （切换协议）请求者已要求服务器切换协议，服务器已确认并准备切换。


** http状态返回代码
    2xx （成功）表示成功处理了请求的状态代码。
代码说明：
200   （成功）  服务器已成功处理了请求。通常，这表示服务器提供了请求的网页。
201   （已创建）  请求成功并且服务器创建了新的资源。
202   （已接受）  服务器已接受请求，但尚未处理。
203   （非授权信息）  服务器已成功处理了请求，但返回的信息可能来自另一来源。
204   （无内容）  服务器成功处理了请求，但没有返回任何内容。
205   （重置内容）服务器成功处理了请求，但没有返回任何内容。
206   （部分内容）  服务器成功处理了部分 GET 请求。


*** http状态返回代码 3xx （重定向）
    表示要完成请求，需要进一步操作。通常，这些状态代码用来重定向。
代码说明：
300   （多种选择）  针对请求，服务器可执行多种操作。服务器可根据请求者 (user agent) 选择一项操作，或提供操作列表供请求者选择。
301   （永久移动）  请求的网页已永久移动到新位置。服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。
302   （临时移动）  服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
303   （查看其他位置）请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。


304   （未修改）自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。
305   （使用代理）请求者只能使用代理访问请求的网页。如果服务器返回此响应，还表示请求者应使用代理。
307   （临时重定向）  服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。


**** http状态返回代码 4xx（请求错误）
    这些状态代码表示请求可能出错，妨碍了服务器的处理。
代码说明：
400   （错误请求）服务器不理解请求的语法。
401   （未授权）请求要求身份验证。对于需要登录的网页，服务器可能返回此响应。
403   （禁止）服务器拒绝请求。
404   （未找到）服务器找不到请求的网页。
405   （方法禁用）禁用请求中指定的方法。
406   （不接受）无法使用请求的内容特性响应请求的网页。
407   （需要代理授权）此状态代码与 401（未授权）类似，但指定请求者应当授权使用代理。
408   （请求超时）  服务器等候请求时发生超时。
409   （冲突）  服务器在完成请求时发生冲突。服务器必须在响应中包含有关冲突的信息。
410   （已删除）  如果请求的资源已永久删除，服务器就会返回此响应。
411   （需要有效长度）服务器不接受不含有效内容长度标头字段的请求。
412   （未满足前提条件）服务器未满足请求者在请求中设置的其中一个前提条件。
413   （请求实体过大）服务器无法处理请求，因为请求实体过大，超出服务器的处理能力。
414   （请求的 URI 过长）请求的 URI（通常为网址）过长，服务器无法处理。
415   （不支持的媒体类型）请求的格式不受请求页面的支持。
416   （请求范围不符合要求）如果页面无法提供请求的范围，则服务器会返回此状态代码。
417   （未满足期望值）服务器未满足"期望"请求标头字段的要求。


***** http状态返回代码 5xx（服务器错误）
    这些状态代码表示服务器在尝试处理请求时发生内部错误。这些错误可能是服务器本身的错误，而不是请求出错。
代码说明：
500   （服务器内部错误）  服务器遇到错误，无法完成请求。
从上往下读，最上面都是抛出异常代码，从上往下，慢慢找出错误产生的原生地方，一般都是最下面是错误出现的地方

501   （尚未实施）服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码。
502   （错误网关）服务器作为网关或代理，从上游服务器收到无效响应。
503   （服务不可用）服务器目前无法使用（由于超载或停机维护）。通常，这只是暂时状态。
504   （网关超时）  服务器作为网关或代理，但是没有及时从上游服务器收到请求。
505   （HTTP 版本不受支持）服务器不支持请求中所用的 HTTP 协议版本。


一些常见的http状态返回代码为：
200 - 服务器成功返回网页
404 - 请求的网页不存在
503 - 服务不可用
```

### 15 请求报文

![image-20201123075848880](.\imgs\image-20201123075848880.png)

### 16 响应报文

![image-20201123080742883](.\imgs\image-20201123080742883.png)

### 17 json数据结构与使用方法

#### 17.1  json的数据结构

json全称: "JavaScript Object Notation",(JavaScript对象表示法),一种基于文本,独立于语言的轻量级数据交换格式。

#### 17.2 JSON规定的格式

​	1)数据在键值对中

​	2)数据由逗号分隔              

​	3)花括号保存对象               

​	4)方括号保存数组

* 在我们python中，就认为成字典+列表组成就是我们的JSON

#### 17.3 json数据的页面响应

* JsonResponse响应json数据

要想返回json数据，就要使用专门的Response对象JSONResponse

```python
from django.shortcuts import render,HttpResponse

方法1

from django.http.response import JsonResponse

def student(requset):
    stu = {'name': "maple", 'age': 12, '爱好': '学习'}
    return JsonResponse(stu,json_dumps_params={'ensure_ascii':False})
    # 只能添加字典

------------------------------------------------------------------------------------------------------

方法2

import json

def student(request):
    stu = {'name': "maple", 'age': 12, '爱好': '学习'}
    s = json.dumps(stu, ensure_ascii=False)
    return HttpResponse(s)
```

* json是一个模块，可以把字典对象变成json字符串

```python
import json
# 我们进行网络传输，必须是字符串或者是二进制数据
# 使用python内置的json模块来完成

stu = {'name': "maple", 'age': 12, '爱好': '学习'}
print(type(stu))
x = json.dumps(stu)
print(x)
# \u 是unicode码值
y = json.dumps(stu,ensure_ascii=False)
print(y)

c = {'name': 'Marry', 'age': 12}

e = json.dumps(c)
print(e, type(e))		# {"name": "Marry", "age": 12} <class 'str'>
f = json.loads(e)
print(f, type(f))		# {'name': 'Marry', 'age': 12} <class 'dict'>
```

#### 17.4 前端

* 将json字符串转换为json对象的方法。在数据传输过程中，json是以文本，即字符串的形式传递的，而JS操作的是JSON对象，所以，JSON对象和JSON字符串之间的相互转换是关键

```python
JSON字符串:
var str1 = '{ "name": "cxh", "sex": "man" }'; 
JSON对象:
var str2 = { "name": "cxh", "sex": "man" };

A. JSON字符串转换为JSON对象

要使用上面的str1，必须使用下面的方法先转化为JSON对象：

//由JSON字符串转换为JSON对象

var obj = eval('(' + str + ')');

或者

var obj = str.parseJSON(); //由JSON字符串转换为JSON对象

或者

var obj = JSON.parse(str); //由JSON字符串转换为JSON对象

然后，就可以这样读取：

Alert(obj.name);

Alert(obj.sex);

特别注意：如果obj本来就是一个JSON对象，那么使用eval（）函数转换后（哪怕是多次转换）还是JSON对象，但是使用parseJSON（）函数处理后会有问题（抛出语法异常）。

B. 可以使用toJSONString()或者全局方法JSON.stringify()将JSON对象转化为JSON字符串。
例如：

var last=obj.toJSONString(); //将JSON对象转化为JSON字符

或者

var last=JSON.stringify(obj); //将JSON对象转化为JSON字符
alert(last)
```

### 18 serializer重写

```python
class UserSerializer(serializers.Serializer):
    # 用serializer重写，不能用ModelViewSet来添加，写上id会报错username不能设为key，因为User表继承的是AbstractUser，其中的username字段必须有unique=True属性设置唯一约束
    username = serializers.CharField()
    password = serializers.CharField()
    mobile = serializers.CharField()
    email = serializers.EmailField()
    token = serializers.CharField(read_only=True)

    def create(self, data):
        username = data.get('username', '')
        password = data.get('password', '')
        mobile = data.get('mobile', '')
        email = data.get('email', '')
        user = User(username=username, email=email, mobile=mobile)
        user.set_password(password)
        user.save()
        # 补充生成记录登录状态的token
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
        payload = jwt_payload_handler(user)
        token = jwt_encode_handler(payload)
        user.token = token
        return user

	def update(self, instance, validated_data):
        # instance 是当前对象obj
        # validated_data 是获取的变量
		instance.mobile = validated_data.get('mobile')
		instance.email = validated_data.get('email')
		instance.save()
		return instance
```

### 19 要注意ModelViewSet的接口路由

* 后端接口

```http
* 查询接口
http://192.168.56.100:1594/user/user/
http://192.168.56.100:1594/user/user/?username='cat'
* 添加接口
http://192.168.56.100:1594/user/user/
* 删除接口(指定用户id删除)
http://192.168.56.100:1594/user/user/id/
* 修改接口(指定用户id修改)
http://192.168.56.100:1594/user/user/id/
```

* 前端访问路径配置

```js
export const get_userlist = P => axios_get('/user/user/', P)     
// 查询所有：获取用户列表
export const search_for = P => axios_get('user/user/?username=' + P.username)   
// 查找指定信息：根据用户名查找指定用户信息并展示
export const add_user = P => axios_post('/user/user/', P)     
// 添加用户信息
export const delete_user = P => axios_delete('/user/user/' + P + '/')           
// 删除指定信息：根据获取到的用户id删除用户信息
export const update_user = P => axios_put('/user/user/'+ P.id +'/', P)    
// 修改指定信息：根绝用户id和提交来的数据修改用户信息
```

### 20 使用 ant-design-vue

```js
// 使用ant-design-vue
import Antd from 'ant-design-vue';
import 'ant-design-vue/dist/antd.css';
Vue.config.productionTip = false
Vue.use(Antd);
```

### 21 ModelViewSet修改问题

* put请求默认是全部修改
* patch请求可以实现局部修改（后端已经封装好了方法）

### 22 Vue嵌套路由

* index.js

```js
import Vue from 'vue'
import Router from 'vue-router'
import Home from '@/components/layout/Home'
// 用于导包路径的配置
const page = name => () => import('@/views/' + name)
Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    // 用户管理模块
    { path: '/',component: Home,name: 'home',
      children: [
        { path: 'usermanage', component: page('user-manage/Index'), name: '用户管理' },
        { path: 'flowconf', component: page('workflow/WorkFlowConf'), name: '模板管理' },
        { path: 'flowtype', component: page('workflow/WorkFlowType'), name: '模板类型管理' },
        { path: 'baidu', component: page('BaiDu'), name: '跳转百度' },
      
      ]
    },
  ]
})
```

* Home页面中写入

```vue
<template>
    <div id="components-layout-demo-basic">
         <a-layout>
              </a-layout-header>
                  <div>
                     <!-- 这里的 router-view 是绑定的路由 -->
                     <router-view></router-view>
                   </div>
              </a-layout-header>
          </a-layout>
    </div>
</template>

<script>
// 导入组件
import LeftMenu from '@/components/layout/LeftMenu'
import Header from '@/components/layout/Header'
export default {
    // 注册组件
    components:{
        LeftMenu,
        Header
    },
}
```

* 绑定左侧菜单LeftMenu

```vue
<template>
  <div>
    <a-menu
      style="width: 256px"
      :default-selected-keys="['1']"
      :default-open-keys="['sub1']"
      :mode="mode"
      :theme="theme"
      @click="handleClick"
    >
    <!-- 必须定义click方法，handleClick是用来跳转路由的，内置毁掉参数e，包括传递上去的key路由地址 -->    
      <a-menu-item key="usermanage">
        <!-- key是路由地址 -->
        用户管理模块
      </a-menu-item>

      <a-menu-item key="baidu">
        百度翻译
      </a-menu-item>
    </a-menu>
  </div>
</template>
<script>
export default {
  data() {
    return {
    };
  },
  methods: {
     handleClick(e) {
      this.current = e.key;
      this.$router.push({path:this.current});
      // click方法默认回调参数中的 key，所以 e.key就是传递来的路由
    },
  },
};
</script>
```

### 23 ant-design-vue 图标大小

* font-size 可以调节图标大小

### 24 v-if v-else控制input框disabled属性

```vue
<template>
    <div>
        <a-modal
        title="Please write now."
        :visible="visible"
        @ok="handleOk"
        @cancel="handleCancel"
        >
        <!-- @ok控制按钮ok -->
        <!-- @cancel控制按钮cancel -->
            <p v-if="userList.id">UserName：
                <a-input 
                style="width:380px;float:right" 
                placeholder="username" 
                v-model="userList.username"
                disabled="disabled"
                ></a-input>    
                <!-- disabled可以用来控制input是否能输入文字 -->
            </p>
            <p v-else>UserName：
                <a-input 
                style="width:380px;float:right" 
                placeholder="username" 
                v-model="userList.username"
                ></a-input>
            </p>
            <br>
            
            <div v-if="userList.id">

            </div>
            
            <div v-else>
            <p>PassWord：
                <a-input
                    v-decorator="[
                        'password',
                        { rules: [{ required: true, message: 'Please input your Password!' }] },
                    ]"
                    type="password"
                    placeholder="Password"
                    style="width:380px;float:right"
                    v-model="userList.password"
                    :disabled = 'false'
                >
                </a-input>
            </p>
        </a-modal>
    </div>
</template>

<script>
export default {
    props:['visible', 'userList'],
    methods: {
        handleOk(e) {
            this.$emit('add')
            // add方法的调用一定要在关闭弹窗上面，否则方法不执行完毕没有办法关闭弹窗        
            this.$emit('update:visible', false)
        },
        handleCancel(e) {
            this.$emit('update:visible', false)
        },
    },
}
</script>

```

### 25 a-table中的分页器&按钮

```vue
 <a-table 
    :columns="要展示的列名称（可以自定义列表）" 
    :data-source="要展示的动态数据表名" 
    :pagination="pagination" 
    :rowKey="(record,index)=>{return index}"
  >
  <!-- 带:的都是属性绑定，不可以更换名字，带 : 就是 js 环境 -->
  <!-- 不写:rowKey="(record,index)=>{return index}"浏览器会发出警告 -->
     <p slot="tags" slot-scope="text,tags,i">
      <!-- 加入操作的按钮！其中tags是某列数据，i是索引 -->
      <a-button @click="delUser(text,tags,i)">删除</a-button>
    </p>
  </a-table>
```

### 26 a-table中获取某一行值的方法

```python
<template>
  <a-table 
    :columns="columns" 
    :data-source="userListGet" 
    :pagination="pagination" 
    :rowKey="(record,index)=>{return index}"
  >
  <!-- 带:的都是属性绑定，不可以更换名字，带 : 就是 js 环境 -->
    <!-- 不写:rowKey="(record,index)=>{return index}"浏览器会发出警告 -->
    <p slot="tags" slot-scope="text,tags,i">
      <!-- 加入操作的按钮！ -->
      <a-button @click="delUser(text,tags,i)">删除</a-button>
    </p>
  </a-table>
</template>

<script>
const columns = [
  {
    title:'操作',
    dataIndex: 'tags',
    key : 'tags',
    width: 100,
    scopedSlots : { customRender: 'tags'}
    // scopedSlots: { customRender: 'tags' },一定不能少不然渲染不了html标签
  }
]

import { delete_user } from '@/http/apis'
export default {
  data() {
    return {
      columns,//这里要返回值哦 
      pagination:{
      pageSize: 4,
      }
    };
  },
  methods:{
    delUser(text,tags,i){
       }     
    },
  }
</script>
```

### 27 控制对话框

```vue
// 定义变量 isDel来控制 confirm，isDel==true执行的就是对话框的 ok，isDel==false执行的就是对话框的 false
const isDel = confirm('你确定要删除' + tags.id)
if(isDel==true){
	delete_user(tags.id).then(
		res=>{
		// 删除回调地址是  http://192.168.56.100:1594/id/
		this.get()
			alert('删除成功啦~')
		})
}else{
	alert('有需要再叫我哈~') 
}
```

### 28 自动跳转到某一网址

```vue
window.location.href = 'https://www.baidu.com/'
```

### 29 Vue报错无法查看对象问题

* 出现问题

```python
Invalid prop: type check failed for prop "dataSource". Expected Array, got Object 
```

* 解决方案

```python
this.userListGet = res.results
```

### 31 Vue+Django-RestFrameWork 实现分页

* 后端代码

  * 实现局部分页

  ```python
  from django.shortcuts import render
  from rest_framework import viewsets
  from rest_framework.pagination import PageNumberPagination
  from rest_framework.response import Response
  from rest_framework.views import APIView
  
  from user.models import User
  from user.serializers import UserSerializer, UserModelSerializer
  
  # 分页（局部）：自定义分页器 局部
  class PageNum(PageNumberPagination):
      page_size = 4                           # 每页显示多少条
      page_size_query_param = 'page_size'     # 查询字符串中代表每页返回数据数量的参数名, 默认值: None
      page_query_param = 'page'               # 查询字符串中代表页码的参数名, 有默认值: page
      max_page_size = None                    # 最大页码数限制
      
  
  class UserViewSet(viewsets.ModelViewSet):
      queryset = User.objects.all()
      serializer_class = UserModelSerializer
      filter_fields = {"username"}
      pagination_class = PageNum  # 注意不是列表（只能有一个分页模式）
  ```

  

* 前端代码

  * Pagination.vue

  ```vue
  <template>
    <div>
      <a-pagination
        show-quick-jumper
        :default-current="2"
        :pageSize = '4'
        :total="count"
        show-less-items
        @change="onChange"
        v-model="current"
      />
        <!-- 一定是change方法，不然不能跳转 -->
    </div>
  </template>
  
  <script>
  export default {
      props:[ 'count' ],
      data() {
          return {
              current:1
          }
      },
      methods: { 
          onChange() {
              this.$emit('getPage', this.current)
      },
      },
      created() {
  
      }
  }
  </script>
  ```

  * Index.vue

  ```vue
  <template>
  <div>
    <div id="components-layout-demo-basic">
          <a-layout-content>
              <TableList
                    :userListGet="userListGet"
                    :userList="userList"
                    @getUser="getUser"
                    @add="add"
               >
                      
               </TableList>
               <Pagination
                    @getPage="getPage" 
                    :count="count"              
                >
      		  </Pagination>
          </a-layout-content>
    </div>
  </div>
  </template>
  
  <script>
  import Pagination from "./components/Pagination"
  
  import { get_userlist } from '@/http/apis';
  export default {
      components:{
          BreadCrumb,
          TableList,
          Search,
          EditForm,
          Pagination
      },
      data() {
          return {
              userListGet:[],
              updateUserList:[],
              // 当前页码
              current:1,
              // 总共的数据多少条
              count:0
  
          }
      },
      methods: {
          getUser(){
            // 获取用户信息列表，父组件传递给子组件
            get_userlist(this.current).then(res=>{
              this.userListGet = res.results
              this.count = res.count
              console.log(this.count)
              console.log(this.userListGet)
            })
          },
          // 获取页码
          getPage(currentChild){
            // 获取到的currentChild是子组件传递过来是第几页
            this.current = currentChild
            console.log(this.current)
            this.getUser()
          }
      },
      created() {
  
      }
  }
  </script>
  
  <style scoped>
  #components-layout-demo-basic {
    text-align: center;
  }
  #components-layout-demo-basic .ant-layout-header,
  #components-layout-demo-basic .ant-layout-footer {
    background: white;
    color: #fff;
  }
  #components-layout-demo-basic .ant-layout-footer {
    line-height: 1.5;
  }
  #components-layout-demo-basic .ant-layout-content {
    background: white;
    color: #fff;
    min-height: 120px;
    line-height: 120px;
  }
  #components-layout-demo-basic > .ant-layout {
    margin-bottom: 48px;
  }
  #components-layout-demo-basic > .ant-layout:last-child {
    margin: 0;
  }
  </style>
  
  ```

### 32 前后端联调分页

* 为了解决前端+后端能同时进行分页的问题，只定义前端分页，后端如果分页就会影响前端数据，如果后端不定义分页器，就会造成后端admin难以管理。

* 后端代码

  * 实现局部分页

  ```python
  from django.shortcuts import render
  from rest_framework import viewsets
  from rest_framework.pagination import PageNumberPagination
  from rest_framework.response import Response
  from rest_framework.views import APIView
  
  from user.models import User
  from user.serializers import UserSerializer, UserModelSerializer
  
  # 分页（局部）：自定义分页器 局部
  class PageNum(PageNumberPagination):
      page_size = 4                           # 每页显示多少条
      page_size_query_param = 'page_size'     # 查询字符串中代表每页返回数据数量的参数名, 默认值: None
      page_query_param = 'page'               # 查询字符串中代表页码的参数名, 有默认值: page
      max_page_size = None                    # 最大页码数限制
      
  
  class UserViewSet(viewsets.ModelViewSet):
      queryset = User.objects.all()
      serializer_class = UserModelSerializer
      filter_fields = {"username"}
      pagination_class = PageNum  # 注意不是列表（只能有一个分页模式）
  ```

  

* 前端代码

  * Pagination.vue

  ```vue
  <template>
    <div>
      <a-pagination
        show-quick-jumper
        :default-current="2"
        :pageSize = '4'
        :total="count"
        show-less-items
        @change="onChange"
        v-model="current"
      />
        <!-- 一定是change方法，不然不能跳转 -->
    </div>
  </template>
  
  <script>
  export default {
      props:[ 'count' ],
      data() {
          return {
              current:1
          }
      },
      methods: { 
          onChange() {
              this.$emit('getPage', this.current)
      },
      },
      created() {
  
      }
  }
  </script>
  ```

  * Index.vue

  ```vue
  <template>
  <div>
    <div id="components-layout-demo-basic">
          <a-layout-content>
              <TableList
                    :userListGet="userListGet"
                    :userList="userList"
                    @getUser="getUser"
                    @add="add"
               >
                      
               </TableList>
               <Pagination
                    @getPage="getPage" 
                    :count="count"              
                >
      		  </Pagination>
          </a-layout-content>
    </div>
  </div>
  </template>
  
  <script>
  import Pagination from "./components/Pagination"
  
  import { get_userlist } from '@/http/apis';
  export default {
      components:{
          BreadCrumb,
          TableList,
          Search,
          EditForm,
          Pagination
      },
      data() {
          return {
              userListGet:[],
              updateUserList:[],
              // 当前页码
              current:1,
              // 总共的数据多少条
              count:0
  
          }
      },
      methods: {
          getUser(){
            // 获取用户信息列表，父组件传递给子组件
            get_userlist(this.current).then(res=>{
              this.userListGet = res.results
              this.count = res.count
              console.log(this.count)
              console.log(this.userListGet)
            })
          },
          // 获取页码
          getPage(currentChild){
            // 获取到的currentChild是子组件传递过来是第几页
            this.current = currentChild
            console.log(this.current)
            this.getUser()
          }
      },
      created() {
  
      }
  }
  </script>
  
  <style scoped>
  #components-layout-demo-basic {
    text-align: center;
  }
  #components-layout-demo-basic .ant-layout-header,
  #components-layout-demo-basic .ant-layout-footer {
    background: white;
    color: #fff;
  }
  #components-layout-demo-basic .ant-layout-footer {
    line-height: 1.5;
  }
  #components-layout-demo-basic .ant-layout-content {
    background: white;
    color: #fff;
    min-height: 120px;
    line-height: 120px;
  }
  #components-layout-demo-basic > .ant-layout {
    margin-bottom: 48px;
  }
  #components-layout-demo-basic > .ant-layout:last-child {
    margin: 0;
  }
  </style>
  
  ```

### 33 利用数据解耦性完善查询接口

```vue
export default {
    components:{
        BreadCrumb,
        TableList,
        Search,
        EditForm,
        Pagination
    },
    data() {
        return {
            roleListGet:[],
            searchList:{
                'zh_name':'',
                'page':1,
                'page_size':4,
                
            },

    },
methods: {
    find(){
              // 根据用户名查找用户信息
              search_for_role(this.searchList).then(res=>{
                  this.getRole()
                  // 数据解耦性！！！查询和查询某个其实可以调用同一个接口！
                  // 查询所有：http://192.168.56.100:1594/?page=1&zh_name=
                  // 查询某个：http://192.168.56.100:1594/?page=1&zh_name=many

              })
            },
    getRole(){
        this.searchList.page = this.current
        // 获取用户信息列表，父组件传递给子组件
        get_rolelist(this.searchList).then(res=>{
        this.roleListGet = res.results
        this.count = res.count
        console.log(this.count)
        console.log(this.roleListGet)
    })
    },
	}
```

### 34 序列化展示外键

* 方法1  新增字段储存名称，新增字段为只读，不影响同一序列化器的添加功能

```python
class FlowConfSerializer(serializers.ModelSerializer):
    flowtype_name = serializers.ReadOnlyField(source='flowtype.name')
    # 第一种序列化展示方法，新增字段flowtype_name储存名称
    class Meta:
        model = FlowConf
        fields = "__all__"
        
'''
{
    "count": 3,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 1,
            "flowtype_name": "财务工单",
            "name": "请假模板1",
            "description": "用于普通请假事宜",
            "customfield": "[{'field_name':'请假时间','type:'1'},{'field_name':'截止时间','type':'1'},{'field_name':'请假原由','type':'2'}]",
            "flowtype": 2
        }
    ]
}
'''
```

* 方法2  替换字段为名称，影响同一序列化器的添加功能

```python
class FlowConfSerializer(serializers.ModelSerializer):
    flowtype = serializers.SerializerMethodField()
    class Meta:
        model = FlowConf
        fields = "__all__"
    def get_flowtype(self, row):
        return row.flowtype.name

'''
{
    "count": 3,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 1,
            "flowtype": "财务工单",
            "name": "请假模板1",
            "description": "用于普通请假事宜",
            "customfield": "[{'field_name':'请假时间','type:'1'},{'field_name':'截止时间','type':'1'},{'field_name':'请假原由','type':'2'}]"
        }
    ]
}
'''
```

### 35 关于 action 属性

#### 35.1 详解

```python
'''
视图集ViewSet
	继承自APIView与ViewSetMixin
    作用也与APIView基本类似，提供了身份认证、权限校验、流量管理等
	ViewSet主要通过继承ViewSetMixin来实现在调用as_view()时传入字典(如{‘get’:’list’})的映射处理工作
	在ViewSet中，没有提供任何动作action方法，需要我们自己实现action方法
	
action的使用
	在视图集中，除了上述默认的方法动作外，还可以添加自定义动作
	只要继承了ViewSetMixin类,路由的配置就发生变化了,只需要写映射即可,视图类的方法中就会有个action

action的属性
在视图集中，我们可以通过action对象属性来获取当前请求视图集时的action动作是哪个
'''
```

* 实例分析

```python
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserModelSerializer
    
	def get_serializer_class(self):
        print('action------->', self.action)
        return UserModelSerializer
```

#### 35.2 获取某条数据

##### 35.2.1 获取某条数据

```http
http://192.168.56.100:1594/user/user/1/
```

* 返回数据
  * 获取某一条数据就不是list类型

```python
action-------> retrieve
```

##### 35.2.2 获取具体数据（查找）

```http
http://192.168.56.100:1594/user/user/?username=魏魏最阔爱
```

* 返回数据
  * 获取到的哪怕一条数据，action动作都是list类型

```python
action-------> list
```

##### 35.2.3 添加数据

```http
http://192.168.56.100:1594/user/user/
```

* 返回数据
  * 添加时create

```python
action-------> create
```

### 36 query和params

* query和params传参的区别

query传参显示参数，params传参不显示参数，params相对于query来说较安全一点

取值方法也有不同：query取值：this.$route.query.XXX || this.$route.params.xxx

query传值页面刷新数据还在，而params传值页面数据消失。

#### 36.1 query

```vue
<!-- 传参组件写法 -->
<router-link :to="{path:'/home', query:{'pid':item.id}}"</router-link>

<!-- 写在方法里 -->
jump(flowConf){
	this.$router.push(
		{
			path:'flowconfform',
			// 如果是path:'flowconfform/',多了/就会无法返回
			'query':{
			'name':flowConf.name,
			'id':flowConf.id
		}
	}
	)
}
```



#### 36.2 params

```python

```

### 37 循环之双向绑定+添加自定义字段

* 父组件

```vue
<template>
    <div>
        <div>
             <a-row>
            <a-col :span="19" style="height:500px">
                <CreateForm
                   :msg="flowConf" 
                ></CreateForm>
            </a-col>
            <a-col :span="5" style="height:500px;background:yellow">
                col-12
            </a-col>
            </a-row>
            <a-row>
            <a-col :span="24" style="height:160px">
                <Check
                    :flowConf="flowConf"
                ></Check>
            </a-col>
            </a-row>
        </div>
    </div>
</template>

<script>
import Check from '@/views/flowconfform-manage/components/Check'
import CreateForm from '@/views/flowconfform-manage/components/CreateForm'
export default {
    components:{
        Check,
        CreateForm
    },
    data() {
        return {
            flowConf:this.$route.query.flowconf,
        }
    },
    methods: {
        changeJson(){
            const textJson = this.flowConf.customfield
            console.log('自定义字段', textJson)
            const textJsonChange = JSON.parse(textJson)
            console.log('转换后自定义字段', textJsonChange)
            this.flowConf.customfield = textJsonChange
            console.log(this.flowConf)
        }

    },
    created() {
    this.changeJson()
    }
}
</script>

<style scoped>

</style>
```

* 子组件

```vue
<template>
    <div><h1>{{ msg.customfield }}</h1>
         <a-form-model :label-col="labelCol" :wrapper-col="wrapperCol">
            <a-form-model-item v-for="(select,item) in msg.customfield.field_list" :key="item">
            
            <p v-if="select.field_type==='input'" style="text-align:left">{{select.verbos_name}}:&emsp;
                <a-input v-model="select.value" style="width:200px" @change="giveValue"/>
            </p>
            
            <p v-show="select.field_type==='select'" style="text-align:left">
                {{select.verbos_name}}:&emsp;
                <a-select v-model="select.value" placeholder="please select your option" style="width:200px" @change="giveValue">
                    <a-select-option v-for="s in select.field_datasource" :key="s.value" :value="s.value">
                    {{ s.label }}
                    </a-select-option>
                </a-select>
            </p>

            <p v-show="select.field_type=='textarea'" style="text-align:left">
                {{select.verbos_name}}:
                <a-input v-model="select.value" type="textarea" style="width:700px;height:150px" @change="giveValue"/>
            </p>
            </a-form-model-item>
            <a-form-model-item :wrapper-col="{ span: 14, offset: 4 }">
                <a-button type="primary" @click="onSubmit">
                提交
                </a-button>
                <a-button style="margin-left: 10px;">
                取消
                </a-button>
            </a-form-model-item>
        </a-form-model>
    </div>
</template>
<script>
export default {
  props:[ 'msg' ],
  data() {
    return {  
      labelCol: { span: 4 },
      wrapperCol: { span: 14 },
      form: {
        name: '',
        region: undefined,
        date1: undefined,
        delivery: false,
        type: [],
        resource: '',
        desc: '',        
      },
      field_datasource:{},
    };
  },
  methods: {
    onSubmit() {
      console.log('submit!', this.form);
    },  
    // 老师的方法
    // giveValue(){
    //     for(var i in this.msg.customfield.form){
    //     const val = this.getInputValue(i);
    //     this.msg.customfield.form[i] = val;
    //   }       
    // },
    // getInputValue(key){
    //     for(var i=0;i<this.msg.customfield.field_list.length;i=i+1){
    //         const fileld_dic = this.msg.customfield.field_list[i];
    //         const k1 = fileld_dic['name'];
    //         if(key == k1){
    //         return fileld_dic['value']
    //     }
    //   }
    //   return ''
    // }
    giveValue(){
        const form_list = Object.keys(this.msg.customfield.form) 
        // 可以获取到所有的key
        for(var i=0;i<this.msg.customfield.field_list.length;i++){
            form_list.forEach(item => {
                if(this.msg.customfield.field_list[i].name==item){
                    this.msg.customfield.form[item]=this.msg.customfield.field_list[i].value
                }
            });
        }
        return this.msg
    }
  },

};
</script>

<style scoped>
</style>

```

### 38 serializer之元祖类型怎么展示名字

#### 38.1 models字段

```python
class WorkOrderModel(BaseModel):
    status_choices = (
        ('1', '审批中'),
        ('2', '被驳回'),
        ('3', '完成')
    )
    order_status = models.CharField('工单状态', help_text='审批中/被驳回/完成', choices=status_choices, default='1', max_length=30)
```

#### 38.2 serializers写法

```python
class WorkOrderSerializer(serializers.ModelSerializer):
    order_status_name = serializers.SerializerMethodField(required=False)

    class Meta:
        model = WorkOrderModel
        fields = '__all__'

    def get_order_status_name(self, row):
        status_choices = dict(row.status_choices)
        return status_choices[row.order_status]
```

### 39 Vue前端json格式注意事项

* 前端接收到的数据只有在是json字符串格式（双引号）的情况下才能转换成json格式！
* 从数据库获取到的数据默认是单引号！

#### 39.1 Vue把单引号变成双引号的方法

```vue
const data = res.results              
var dic = data.replace(/'/g, '"')
```

#### 39.2 把双引号数据变成json格式

```vue
this.form = JSON.parse(dic)
```

### 40 后端常见问题（filter&get）

```python
obj1 = FlowConf.objects.filter(id=1)			# 这是个列表（嵌套字典形式），是queryset对象
obj2 = FlowConf.objects.get(id=1)  				# 找不到会报错，具体值
obj3 = FlowConf.objects.filter(id=1)[0]			# obj对象（字典形式，.name可以取出具体值）
```

### 41 前端常见问题（登录不上）

* 前端登录不上
  * login页面有问题：登录成功没有设置token（页面就能查看）
  * main.js有问题：查看全局过滤配置，可能没有设置token
  * http/index.js有问题：后端身份验证失败：错误token/token过期

### 42 VUE端v-for循环解决textarea框重复输入的问题

```vue
<template>
   <div>
         <div v-for=" (self, item) in list" :key="item">                         
           <a-textarea
               v-model="value[item]"
           />
   </div>
</template>
<script>
    data() {
        return {
            value:[],
            list:[
                {'id':1},
                {'id':2}
            ]
        }
    },
</script>
```

### 43 自己写的一个权限验证

```python
class SubOrderView(APIView):
    def get(self, request):
        token = request.META.get('HTTP_AUTHORIZATION')
        # 用户id和角色
        user_info = jwt_decode_handler(token[4:])
        user_id = user_info.get('user_id')
        role_info = UserRole.objects.filter(user_id=user_id).values('role_id')
        role_list = []
        for i in role_info:
            role_list.append(i.get('role_id'))
        # 实例化子工单id
        suborder_id = request.query_params.get('id')
        # 审批类型id，1角色审批，2指定人审批
        approve_type_id = SubOrderModel.objects.filter(pk=suborder_id).first().approve_type_id
        if approve_type_id == '1':
            approve_userrole_id = SubOrderModel.objects.filter(pk=suborder_id).first().approve_userrole_id
            if approve_userrole_id in role_list:
                return Response({'msg': '有审批权限', 'code': 200, 'type': 1})
            return Response({'msg': '无审批权限', 'code': 200, 'type': 0})
        elif approve_type_id == '2':
            approve_user = SubOrderModel.objects.filter(pk=suborder_id).first().approve_user
            if user_id == approve_user:
                return Response({'msg': '有审批权限', 'code': 200, 'type': 1})
            return Response({'msg': '无审批权限', 'code': 200, 'type': 0})
```

### 44 JWT生成token

#### 44.1 settings.py

```python
"""
Django settings for opwf project.

Generated by 'django-admin startproject' using Django 2.0.13.

For more information on this file, see
https://docs.djangoproject.com/en/2.0/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/2.0/ref/settings/
"""
import datetime
import os, sys

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.0/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'uorj1ni^mnut@wo@c%)iv)%5=8dxlml4-j0!f3b%4#f*8a5)3t'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ['*']


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'user.apps.UserConfig',
    'workflow.apps.WorkflowConfig',
    'workerorder.apps.WorkerorderConfig',
    # 'jwt',
    # 'rest_framework_jwt',
    # 'rest_framework.authentication'

]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'opwf.urls'
CORS_ORIGIN_ALLOW_ALL = True

CORS_ORIGIN_WHITELIST = (
    'http://127.0.0.1:8080',
    'http://localhost:8080',
)
CORS_ALLOW_CREDENTIALS = True

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'opwf.wsgi.application'


# Database
# https://docs.djangoproject.com/en/2.0/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'opwf_db',
        'USER': 'root',
        'PASSWORD': '1',
        'HOST': '127.0.0.1',
        'PORT': '3306'
    }
}


# Password validation
# https://docs.djangoproject.com/en/2.0/ref/settings/#auth-password-validators

REST_FRAMEWORK = {
    # 文档报错： AttributeError: ‘AutoSchema’ object has no attribute ‘get_link’
    # 用下面的设置可以解决
    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.AutoSchema',
    # 默认设置是:
    # 'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.openapi.AutoSchema',

    # 异常处理器
    # 'EXCEPTION_HANDLER': 'user.utils.exception_handler',

    # Base API policies    　　默认渲染器类
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
    # 默认解析器类
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.FormParser',
        'rest_framework.parsers.MultiPartParser'
    ],
    # 1.认证器（全局）
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # 在 DRF中配置JWT认证
        # 'rest_framework.authentication.SessionAuthentication',  # 使用session时的认证器
        # 'rest_framework.authentication.BasicAuthentication'  # 提交表单时的认证器
    ],

    # 2.权限配置（全局）： 顺序靠上的严格
    'DEFAULT_PERMISSION_CLASSES': [
        # 'rest_framework.permissions.IsAdminUser',  # 管理员可以访问
        # 'rest_framework.permissions.IsAuthenticated',  # 认证用户可以访问
        # 'rest_framework.permissions.IsAuthenticatedOrReadOnly',  # 认证用户可以访问, 否则只能读取
        'rest_framework.permissions.AllowAny',  # 所有用户都可以访问
    ],
    # 3.限流（防爬虫）
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    # 3.1限流策略
    # 'DEFAULT_THROTTLE_RATES': {
    #     'user': '100/hour',  # 认证用户每小时100次
    #     'anon': '300/day',  # 未认证用户每天能访问3次
    # },

    'DEFAULT_CONTENT_NEGOTIATION_CLASS': 'rest_framework.negotiation.DefaultContentNegotiation',
    'DEFAULT_METADATA_CLASS': 'rest_framework.metadata.SimpleMetadata',
    'DEFAULT_VERSIONING_CLASS': None,

    # 4.分页（全局）：全局分页器, 例如 省市区的数据自定义分页器, 不需要分页
    # 'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    # # 每页返回数量
    # 'PAGE_SIZE': 3,
    # 5.过滤器后端
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        # 'django_filters.rest_framework.backends.DjangoFilterBackend', 包路径有变化
    ],

    # 5.1过滤排序（全局）：Filtering 过滤排序
    'SEARCH_PARAM': 'search',
    'ORDERING_PARAM': 'ordering',

    'NUM_PROXIES': None,

    # 6.版本控制：Versioning  接口版本控制
    'DEFAULT_VERSION': None,
    'ALLOWED_VERSIONS': None,
    'VERSION_PARAM': 'version',

    # Authentication  认证
    # 未认证用户使用的用户类型
    'UNAUTHENTICATED_USER': 'django.contrib.auth.models.AnonymousUser',
    # 未认证用户使用的Token值
    'UNAUTHENTICATED_TOKEN': None,

    # View configuration
    'VIEW_NAME_FUNCTION': 'rest_framework.views.get_view_name',
    'VIEW_DESCRIPTION_FUNCTION': 'rest_framework.views.get_view_description',

    'NON_FIELD_ERRORS_KEY': 'non_field_errors',

    # Testing
    'TEST_REQUEST_RENDERER_CLASSES': [
        'rest_framework.renderers.MultiPartRenderer',
        'rest_framework.renderers.JSONRenderer'
    ],
    'TEST_REQUEST_DEFAULT_FORMAT': 'multipart',

    # Hyperlink settings
    'URL_FORMAT_OVERRIDE': 'format',
    'FORMAT_SUFFIX_KWARG': 'format',
    'URL_FIELD_NAME': 'url',

    # Encoding
    'UNICODE_JSON': True,
    'COMPACT_JSON': True,
    'STRICT_JSON': True,
    'COERCE_DECIMAL_TO_STRING': True,
    'UPLOADED_FILES_USE_URL': True,

    # Browseable API
    'HTML_SELECT_CUTOFF': 1000,
    'HTML_SELECT_CUTOFF_TEXT': "More than {count} items...",

    # Schemas
    'SCHEMA_COERCE_PATH_PK': True,
    'SCHEMA_COERCE_METHOD_NAMES': {
        'retrieve': 'read',
        'destroy': 'delete'
    },

    # 'Access-Control-Allow-Origin':'http://localhost:8080',
    # 'Access-Control-Allow-Credentials': True

}

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/2.0/topics/i18n/

LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = False


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.0/howto/static-files/

STATIC_URL = '/static/'
AUTH_USER_MODEL = 'user.User'

# jwt载荷中的有效期设置
JWT_AUTH = {
    # 1.token前缀：headers中 Authorization 值的前缀
    'JWT_AUTH_HEADER_PREFIX': 'JWT',
    # 2.token有效期：一天有效
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
    # 3.刷新token：允许使用旧的token换新token
    'JWT_ALLOW_REFRESH': True,
    # 4.token有效期：token在24小时内过期, 可续期token
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(hours=24),
    # 5.自定义JWT载荷信息：自定义返回格式，需要手工创建
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'user.utils.jwt_response_payload_handler',
}
```

#### 44.2 utils.py

```python
# -*- coding: utf-8 -*-
def jwt_response_payload_handler(token, user=None, request=None, role=None):
    """
        自定义jwt认证成功返回数据
        :token 返回的jwt
        :user 当前登录的用户信息[对象]
        :request 当前本次客户端提交过来的数据
        :role 角色
    """
    if user.first_name:
        name = user.first_name
    else:
        name = user.username
        return {
            'authenticated': 'true',
             'id': user.id,
             "role": role,
             'name': name,
             'username': user.username,
             'email': user.email,
             'token': token,
        }
```

#### 44.3 models.py

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    '''
    username：用户名
    password：密码
    mobile：手机号
    email：邮箱
    '''
    username = models.CharField(max_length=30, unique=True)
    # 写上 unique=True 就可以指定唯一，验证字段的时候自动验证
    password = models.CharField(max_length=256)
    mobile = models.CharField(max_length=11)
    email = models.CharField(max_length=30)
    token = models.CharField(max_length=256, default='')
    weixin = models.CharField(max_length=30, null=True)
    date_joined = models.DateField(auto_now_add=True)

    class Meta:
        db_table = 'user_user'
        verbose_name = '用户表'
        verbose_name_plural = verbose_name
```

#### 44.4 serializers.py

```python
# -*- coding: utf-8 -*-
from rest_framework import serializers
from rest_framework_jwt.serializers import jwt_payload_handler
from rest_framework_jwt.settings import api_settings
from user.models import User
class UserSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    username = serializers.CharField()
    password = serializers.CharField()
    mobile = serializers.CharField()
    email = serializers.EmailField()
    weixin = serializers.CharField()
    token = serializers.CharField(read_only=True)

    def create(self, data):
        username = data.get('username', '')
        password = data.get('password', '')
        mobile = data.get('mobile', '')
        email = data.get('email', '')
        weixin = data.get('weixin', '')

        user = User(username=username, email=email, mobile=mobile, weixin=weixin)
        user.set_password(password)
        user.save()
        # 补充生成记录登录状态的token
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
        payload = jwt_payload_handler(user)
        token = jwt_encode_handler(payload)
        user.token = token
        return user
```

#### 44.5 urls.py

```python
from django.urls import path
from rest_framework.routers import DefaultRouter
from rest_framework_jwt.views import obtain_jwt_token

from user import views

router = DefaultRouter()
router.register(r'user', views.UserViewSet)
router.register(r'user_get', views.UserGetViewSet)
router.register(r'role', views.RoleViewSet)
router.register(r'role_get', views.RoleGetViewSet)

urlpatterns = [
    path('login/', obtain_jwt_token),
    path('register/', views.RegisterView.as_view()),
]
```

### 45 token解码

```python
# 如果是APIView获取的，在VUE中定义的，可以使用以下方式解密

from rest_framework_jwt.utils import jwt_decode_handler

def get(self, request):
        token = request.META.get('HTTP_AUTHORIZATION')
        # 用户id和角色
        user_info = jwt_decode_handler(token[4:])
```

### 46 serializer 关于查询

* 以工单表和工单子表为例

#### 46.1 models.py

```python
from django.db import models

# Create your models here.
from utils.MyBaseModel import BaseModel
from user.models import User
from workflow.models import FlowConf



class WorkOrderModel(BaseModel):
    '''
    flowconf：工单名称（一对多，FlowConf）
    create_user：工单创建用户（一对多，User表）
    create_ts：创建时间
    order_status：工单状态（审批中/被驳回/完成）
    description：描述（text，存储用户工单描述）
    '''
    status_choices = (
        ('1', '审批中'),
        ('2', '被驳回'),
        ('3', '完成')
    )
    flowconf = models.ForeignKey(FlowConf, on_delete=models.CASCADE, null=True)
    create_user = models.ForeignKey(User, on_delete=models.CASCADE)
    order_status = models.CharField('工单状态', help_text='审批中/被驳回/完成', choices=status_choices, default='1', max_length=30)
    parameter = models.TextField(default='{}')
    # description = models.TextField('描述', help_text='存储用户工单描述')

    class Meta:
        db_table = 'workerorder_workerorder'
        verbose_name = '实例化工单'
        verbose_name_plural = verbose_name



class SubOrderModel(BaseModel):
    '''
    mainorder：一对多（WorkOrder）实例化工单相连
    approve_user：一对多（内置User表）审批人
    approbe_user_role：审批角色
    approve_userrole_id：审批角色id
    sequence_number：审批序号
    approve_ts：审批时间
    action_status：审批状态（待审批/通过/拒绝/退回）
    suborder_status：子任务状态（待处理/已经处理/待上一节点处理）
    approve_text：审批意见
    approve_type_id: 审批类型
    '''
    action_status_choices = (
        ('1', '待审批'),
        ('2', '通过'),
        ('3', '拒绝'),
        ('4', '退回')
    )
    suborder_status_choices = (
        ('1', '待处理'),
        ('2', '已经处理'),
        ('3', '待上一节点处理')
    )
    mainorder = models.ForeignKey(WorkOrderModel, on_delete=models.CASCADE)
    approve_user = models.ForeignKey(User, null=True, blank=True, on_delete=models.SET_NULL, default='')
    approbe_user_role = models.CharField('审批角色', max_length=30, null=True)
    approve_userrole_id = models.IntegerField('审批角色id')
    sequence_number = models.IntegerField('审批序号')

    approve_ts = models.DateTimeField('审批时间', auto_now_add=True)
    action_status = models.CharField('审批状态', help_text='待审批/通过/拒绝/退回', choices=action_status_choices, default='1', max_length=30)
    suborder_status = models.CharField('子任务状态', help_text='待处理/已经处理/待上一节点处理', choices=suborder_status_choices, default='1', max_length=30)
    approve_text = models.TextField('审批意见')
    type_choice = (
        ('1', '角色审批'),
        ('2', '指定人员审批')
    )
    approve_type_id = models.CharField(max_length=30, choices=type_choice)

    class Meta:
        db_table = 'workerorder_suborder'
        verbose_name = '实例化子工单'
        verbose_name_plural = verbose_name
```

#### 46.2 serializer.py

```python
from rest_framework import serializers
from workerorder.models import WorkOrderModel, SubOrderModel
from workflow.models import FlowConf


class WorkOrderSerializer(serializers.ModelSerializer):
    order_status_name = serializers.SerializerMethodField(required=False)
    create_user_name = serializers.SerializerMethodField(required=False)
    flowconf_name = serializers.CharField(required=False, source='flowconf.name')

    class Meta:
        model = WorkOrderModel
        fields = '__all__'

    def get_order_status_name(self, row):
        status_choices = dict(row.status_choices)
        return status_choices[row.order_status]

    def get_create_user_name(self, row):
        # print(type(row.create_user.username))
        # str
        return row.create_user.username


class SubOrderSerializer(serializers.ModelSerializer):
    approve_user_name = serializers.SerializerMethodField(required=False)
    action_status_name = serializers.SerializerMethodField(required=False)
    suborder_status_name = serializers.SerializerMethodField(required=False)
    flowconf_name = serializers.SerializerMethodField(required=False)
    class Meta:
        model = SubOrderModel
        fields = "__all__"

    def get_action_status_name(self, row):
        return dict(row.action_status_choices)[row.action_status]

    def get_suborder_status_name(self, row):
        return dict(row.suborder_status_choices)[row.suborder_status]

    def get_approve_user_name(self, row):
        if row.approve_user:
            return row.approve_user.username
        else:
            return ''
    def get_flowconf_name(self, row):
        return row.mainorder.flowconf.name
```

### 47 Model字段类型

#### 47.1 常用字段类型

```python
from django.db import models

class UserGroup(models.Model):
    uid = models.AutoField(primary_key=True)

    name = models.CharField(max_length=32,null=True, blank=True)
    email = models.EmailField(max_length=32)
    text = models.TextField()

    ctime = models.DateTimeField(auto_now_add=True)      # 只有添加时才会更新时间
    uptime = models.DateTimeField(auto_now=True)         # 只要修改就会更新时间

    ip1 = models.IPAddressField()                  # 字符串类型，Django Admin以及ModelForm中提供验证 IPV4 机制
    ip2 = models.GenericIPAddressField()           # 字符串类型，Django Admin以及ModelForm中提供验证 Ipv4和Ipv6

    active = models.BooleanField(default=True)

    data01 = models.DateTimeField()                      # 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ
    data02 = models.DateField()                          # 日期格式      YYYY-MM-DD
    data03 = models.TimeField()                          # 时间格式      HH:MM[:ss[.uuuuuu]]

    age = models.PositiveIntegerField()           # 正小整数 0 ～ 32767
    balance = models.SmallIntegerField()          # 小整数 -32768 ～ 32767
    money = models.PositiveIntegerField()         # 正整数 0 ～ 2147483647
    bignum = models.BigIntegerField()             # 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807
    
    user_type_choices = (
        (1, "超级用户"),
        (2, "普通用户"),
        (3, "普普通用户"),
    )
    user_type_id = models.IntegerField(choices=user_type_choices, default=1)

```

#### 47.2 不常用字段类型

```python
URLField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证 URL

    SlugField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证支持 字母、数字、下划线、连接符（减号）

    CommaSeparatedIntegerField(CharField)
        - 字符串类型，格式必须为逗号分割的数字

    UUIDField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供对UUID格式的验证

    FilePathField(Field)
        - 字符串，Django Admin以及ModelForm中提供读取文件夹下文件的功能
        - 参数：
                path,                      文件夹路径
                match=None,                正则匹配
                recursive=False,           递归下面的文件夹
                allow_files=True,          允许文件
                allow_folders=False,       允许文件夹

    FileField(Field)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

    ImageField(FileField)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
            width_field=None,   上传图片的高度保存的数据库字段名（字符串）
            height_field=None   上传图片的宽度保存的数据库字段名（字符串）


    DurationField(Field)
        - 长整数，时间间隔，数据库中按照bigint存储，ORM中获取的值为datetime.timedelta类型

    FloatField(Field)
        - 浮点型

    DecimalField(Field)
        - 10进制小数
        - 参数：
            max_digits，小数总长度
            decimal_places，小数位长度

    BinaryField(Field)
        - 二进制类型
```

### 48 一对多ForeignKey可选参数

```python
1、to,                                          # 要进行关联的表名
2、to_field=None,                               # 要关联的表中的字段名称
3、on_delete=None,                              # 当删除关联表中的数据时，当前表与其关联的行的行为
            - models.CASCADE                    # ，删除关联数据，与之关联也删除
            - models.DO_NOTHING                 # ，删除关联数据，引发错误IntegrityError
            - models.PROTECT                    # ，删除关联数据，引发错误ProtectedError
            - models.SET_NULL                   # ，删除关联数据，与之关联的值设置为null（前提FK字段需要设置为可空）
            - models.SET_DEFAULT                # ，删除关联数据，与之关联的值设置为默认值（前提FK字段需要设置默认值）
            - models.SET                        # ，删除关联数据，
4、related_name=None,                           # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
                                                # 在做自关联时必须指定此字段，防止查找冲突
5、delated_query_name=None,                     # 反向操作时，使用的连接前缀，用于替换【表名】
                                                # 如：models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
6、limit_choices_to=None,                       # 在Admin或ModelForm中显示关联数据时，提供的条件：
            - limit_choices_to={'nid__gt': 5}
            - limit_choices_to=lambda : {'nid__gt': 5}
7、db_constraint=True                           # 是否在数据库中创建外键约束
8、parent_link=False                            # 在Admin中是否显示关联数据
一对多ForeignKey可选参数
```

### 49 Django 一对多表结构操作

#### 49.1 一对多基本增删改查

* models.py

```python
from django.db import models

class UserInfo(models.Model):
    name = models.CharField(max_length=64,unique=True)
    ut = models.ForeignKey(to='UserType')

class UserType(models.Model):
    type_name = models.CharField(max_length=64,unique=True)

```

* views.py

```python
from django.shortcuts import render,HttpResponse
from app01 import models

def orm(request):
    # 1 创建
    # 创建数据方法一
    models.UserInfo.objects.create(name='root', ut_id=2)
    # 创建数据方法二
    obj = models.UserInfo(name='root', ut_id=2)
    obj.save()
    # 创建数据库方法三(传入字典必须在字典前加两个星号)
    dic = {'name': 'root', 'ut_id': 2}
    models.UserInfo.objects.create(**dic)

    # 2 删除
    # models.UserInfo.objects.all().delete()  # 删除所有
    models.UserInfo.objects.filter(name='root').delete()  # 删除指定

    # 3 更新
    # models.UserInfo.objects.all().update(ut_id=1)
    # models.UserInfo.objects.filter(name='zhangsan').update(ut_id=4)

    # 4.1 正向查找 user_obj.ut.type_name
    print( models.UserInfo.objects.get(name='zhangsan').ut.type_name )
    print( models.UserInfo.objects.filter(ut__type_name='student') )

    # 4.2 反向查找 type_obj.userinfo_set.all()
    print( models.UserType.objects.get(type_name='student').userinfo_set.all() )
    print( models.UserType.objects.get(type_name='student').userinfo_set.filter(name='zhangsan') )

    return HttpResponse('orm')

views.py
```

#### 49.2 一对多更多查询

* 一对多创建表

```python
from django.db import models

class UserType(models.Model):
    user_type_name = models.CharField(max_length=32)
    def __str__(self):
        return self.user_type_name            #只有加上这个，Django admin才会显示表名

class User(models.Model):
    username = models.CharField(max_length=32)
    pwd = models.CharField(max_length=64)
    ut = models.ForeignKey(
        to='UserType',
        to_field='id',

        # 1、反向操作时，使用的连接前缀，用于替换【表名】
        # 如： models.UserGroup.objects.filter(a__字段名=1).values('a__字段名')
        related_query_name='a',

        #2、反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.b_set.all()
        # 使用时查找报错
        # related_name='b',
    )
```

* 一对多正向反向查找

```python
from django.shortcuts import HttpResponse
from app01 import models

def orm(request):
    # 1 正向查找
    #1.1 正向查找user表用户名
    print(models.User.objects.get(username='zhangsan').username)           # zhangsan

    #1.2 正向跨表查找用户类型
    print(models.User.objects.get(username='zhangsan').ut.user_type_name)  # student

    #1.3 双下划线正向跨表正向查找
    print( models.User.objects.all().values('ut__user_type_name','username') )


    # 2 反向查找
    # 2.1：【表名_set】，反向查找user表中用户类型为student 的所有用户
    print( models.UserType.objects.get(user_type_name='student').user_set.all() )           # [<User: lisi>, <User: wangwu>]

    # 2.2：【a__字段名】反向查找user表中张三在UserType表中的类型：（[<UserType: teacher>]）
    print( models.UserType.objects.filter(a__username='zhangsan') )                         # student
    # 这里的a是user表的ForeignKey字段的参数：related_query_name='a'

    # 2.3: 双下划线跨表反向查找
    print( models.UserType.objects.all().values('a__username', 'user_type_name') )


    # 3 自动创建User表和UserType表中的数据
    '''
    username = [{'username':'zhangsan','pwd':'123','ut_id':'1'},
                {'username':'lisi','pwd':'123','ut_id':'1'},
                {'username':'wangwu','pwd':'123','ut_id':'1'},]

    user_type = [{'user_type_name':'student'},{'user_type_name':'teacher'},]

    for type_dic in user_type:
        models.UserType.objects.create(**type_dic)

    for user_dic in username:
        models.User.objects.create(**user_dic)
    '''
    return HttpResponse('orm')
```

* 一对多使用values和values_list结合双下划线跨表查询

```python
from django.shortcuts import HttpResponse
from app01 import models

def orm(request):
    # 第一种：values-----获取的内部是字典  拿固定列数
    # 1.1 正向查找： 使用ForeignKey字段名ut结合双下划线查询
    models.User.objects.filter(username='zhangsan').values('username', 'ut__user_type_name')

    # 1.2 向查找： 使用ForeignKey的related_query_name='a',的字段
    models.UserType.objects.all().values('user_type_name', 'a__username')


    # 第二种：values_list-----获取的是元组  拿固定列数
    # 1.1 正向查找： 使用ForeignKey字段名ut结合双下划线查询
    stus = models.User.objects.filter(username='zhangsan').values_list('username', 'ut__user_type_name')

    # 1.2 反向查找： 使用ForeignKey的related_query_name='a',的字段
    utype = models.UserType.objects.all().values_list('user_type_name', 'a__username')


    # 3 自动创建User表和UserType表中的数据
    '''
    username = [{'username':'zhangsan','pwd':'123','ut_id':'1'},
                {'username':'lisi','pwd':'123','ut_id':'1'},
                {'username':'wangwu','pwd':'123','ut_id':'1'},]

    user_type = [{'user_type_name':'student'},{'user_type_name':'teacher'},]

    for type_dic in user_type:
        models.UserType.objects.create(**type_dic)

    for user_dic in username:
        models.User.objects.create(**user_dic)
    '''

    return HttpResponse('orm')
```

### 50 Django 多对多表结构操作

#### 50.1 创建并操作多对多表

##### 50.1.1 m2m字段创建

* 自己不创建第三张关系表，有m2m字段: 根据queryset对象增删改查（推荐）

```python
from django.shortcuts import HttpResponse
from app01 import models

def orm(request):
    user_info_obj = models.UserInfo.objects.get(username='zhangsan')
    user_info_objs = models.UserInfo.objects.all()

    group_obj = models.UserGroup.objects.get(group_name='group_python')
    group_objs = models.UserGroup.objects.all()

    # 添加: 正向
    group_obj.user_info.add(user_info_obj)
    group_obj.user_info.add(*user_info_objs)
    # 删除：正向
    group_obj.user_info.remove(user_info_obj)
    group_obj.user_info.remove(*user_info_objs)

    # 添加: 反向
    user_info_obj.usergroup_set.add(group_obj)
    user_info_obj.usergroup_set.add(*group_objs)
    # 删除：反向
    user_info_obj.usergroup_set.remove(group_obj)
    user_info_obj.usergroup_set.remove(*group_objs)

    # 查找：正向
    print(group_obj.user_info.all())                                # 查找group_python组中所有用户
    print(group_obj.user_info.all().filter(username='zhangsan'))
    # 查找：反向
    print(user_info_obj.usergroup_set.all())                        # 查找用户zhangsan属于那些组
    print(user_info_obj.usergroup_set.all().filter(group_name='group_python'))


    # 双下划线 正向、反向查找
    # 正向：从用户组表中查找zhangsan属于哪个用户组：[<UserGroup: group_python>]
    print( models.UserGroup.objects.filter(user_info__username='zhangsan'))

    # 反向：从用户表中查询group_python组中有哪些用户：related_query_name='m2m'
    print( models.UserInfo.objects.filter(m2m__group_name='group_python'))


    # 自动创建UserInfo表和UserGroup表中的数据
    '''
    user_list = [{'username':'zhangsan'},
                {'username':'lisi'},
                {'username':'wangwu'},]
    group_list = [{'group_name':'group_python'},
               {'group_name':'group_linux'},
               {'group_name':'group_mysql'},]

    for c in user_list:
        models.UserInfo.objects.create(**c)

    for l in group_list:
        models.UserGroup.objects.create(**l)
    '''

    return HttpResponse('orm')
```

##### 50.1.2 手动创建第三张表

```python
from django.db import models

#表1：主机表
class Host(models.Model):
    nid = models.AutoField(primary_key=True)
    hostname = models.CharField(max_length=32,db_index=True)

#表2：应用表
class Application(models.Model):
    name = models.CharField(max_length=32)

#表3：自定义第三张关联表
class HostToApp(models.Model):
    hobj = models.ForeignKey(to="Host",to_field="nid")
    aobj = models.ForeignKey(to='Application',to_field='id')

# 向第三张表插入数据，建立多对多外键关联
HostToApp.objects.create(hobj_id=1,aobj_id=2)
```

##### 50.1.3 m2m创建（id查询）

* 自己不创建第三张关系表，有m2m字段，根据数字id增删改查

```python
from django.db import models

class Host(models.Model):
    hostname = models.CharField(max_length=32,db_index=True)

class Group(models.Model):
    group_name = models.CharField(max_length=32)
    m2m = models.ManyToManyField("Host")

    
```

* 查询

```python
from django.shortcuts import HttpResponse
from app01 import models

def orm(request):
    # 使用间接方法对第三张表操作
    obj = models.Group.objects.get(id=1)

    # 1、添加
    obj.m2m.add(1)           # 在第三张表中增加一个条目(1,1)
    obj.m2m.add(2, 3)        # 在第三张表中增加条目（1,2）（1,3）两条关系
    obj.m2m.add(*[1,3])        # 在第三张表中增加条目（1,2）（1,3）两条关系

    # 2、删除
    obj.m2m.remove(1)             # 删除第三张表中的（1,1）条目
    obj.m2m.remove(2, 3)          # 删除第三张表中的（1,2）（1,3）条目
    obj.m2m.remove(*[1, 2, 3])    # 删除第三张表中的（1,1）（1,2）（1,3）条目

    # 3、清空
    obj.m2m.clear()                 # 删除第三张表中application条目等于1的所有条目

    # 4 更新
    obj.m2m.set([1, 2,])             # 第三张表中会删除所有条目，然后创建（1,1）（1,2）条目

    # 5 查找
    print( obj.m2m.all() )           # 等价于 models.UserInfo.objects.all()

    # 6 反向查找： 双下划线
    hosts = models.Group.objects.filter(m2m__id=1)         # 在Host表中id=1的主机同时属于那些组


    # 自动创建Host表和Group表中的数据
    '''
    hostname = [{'hostname':'zhangsan'},
                {'hostname':'lisi'},
                {'hostname':'wangwu'},]
    group_name = [{'group_name':'DBA'},{'group_name':'public'},]

    for h in hostname:
        models.Host.objects.create(**h)
    for u in group_name:
        models.Group.objects.create(**u)
    '''

    return HttpResponse('orm')
```

#### 50.2 values和values_list结合双下划线跨表查询

```python
from django.shortcuts import HttpResponse
from app01 import models

def orm(request):
    # 第一种：values-----获取的内部是字典,拿固定列数
    # 1.1 正向查找： 使用ManyToManyField字段名user_info结合双下划线查询
    models.UserGroup.objects.filter(group_name='group_python').values('group_name', 'user_info__username')

    # 1.2 反向查找： 使用ManyToManyField的related_query_name='m2m',的字段
    models.UserInfo.objects.filter(username='zhangsan').values('username', 'm2m__group_name')


    # 第二种：values_list-----获取的是元组  拿固定列数
    # 2.1 正向查找： 使用ManyToManyField字段名user_info结合双下划线查询
    models.UserGroup.objects.filter(group_name='group_python').values_list('group_name', 'user_info__username')

    # 2.2 反向查找： 使用ManyToManyField的related_query_name='m2m',的字段
    lesson = models.UserInfo.objects.filter(username='zhangsan').values_list('username', 'm2m__group_name')



    # 自动创建UserInfo表和UserGroup表中的数据
    '''
    # user_info_obj = models.UserInfo.objects.get(username='lisi')
    # user_info_objs = models.UserInfo.objects.all()
    #
    # group_obj = models.UserGroup.objects.get(group_name='group_python')
    # group_objs = models.UserGroup.objects.all()
    #
    # group_obj.user_info.add(*user_info_objs)
    # user_info_obj.usergroup_set.add(*group_objs)

    user_list = [{'username':'zhangsan'},
                {'username':'lisi'},
                {'username':'wangwu'},]
    group_list = [{'group_name':'group_python'},
               {'group_name':'group_linux'},
               {'group_name':'group_mysql'},]

    for c in user_list:
        models.UserInfo.objects.create(**c)

    for l in group_list:
        models.UserGroup.objects.create(**l)
    '''

    return HttpResponse('orm')
```

#### 50.3 多对多时ManyToManyField可以添加的参数

```python
1、to,                        # 要进行关联的表名
2、related_name=None,         # 反向操作时，使用的字段名，用于代替 【表名_set】如： obj.表名_set.all()
3、related_query_name=None,   # 反向操作时，使用的连接前缀，用于替换【表名】     
                              # 如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
4、limit_choices_to=None,     # 在Admin或ModelForm中显示关联数据时，提供的条件：
                              # - limit_choices_to={'nid__gt': 5}
5、symmetrical=None,          # 用于多对多自关联，symmetrical用于指定内部是否创建反向操作字段
6、through=None,              # 自定义第三张表时，使用字段用于指定关系表
7、through_fields=None,       # 自定义第三张表时，使用字段用于指定关系表中那些字段做多对多关系表
8、db_constraint=True,        # 是否在数据库中创建外键约束
   db_table=None,             # 默认创建第三张表时，数据库中表的名称
```