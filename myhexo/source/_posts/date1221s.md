---
title: celery定时任务+redis有序集合实现实时访问
date: 2020-09-10 13:14:21 +8000
tags: [celery,redis,实时访问]
---

### 1 概述

#### 1.1 项目理解

* 数据冗余

数据冗余是指数据之间的重复，也可以说是同一数据存储在不同数据文件中的现象。可以说增加数据的独立性和减少数据冗余是企业范围信息资源管理和大规模信息系统获得成功的前提条件。

数据冗余或者信息冗余是生产、生活所必然存在的行为，没有好与不好的总体倾向。

* 如何解决重复ip？

去重，可以采用唯一索引。采用实时数据，mysql，mongofb，不适合实现这次的效果。采用redis数据库，可以利用其五大数据类型中的有序集合。

* 如何理解可变，不可变？

唯一性大于有序就是可变类型；元祖，数字，字符串，不可变集合是不可变类型。但实质上，集合是可变的。

* 集合底层实现？

哈希实现，字典实现，一种没有value的字典，集合中的，expire不能设置集合中某个键值对的过期时间！

* 对于时间戳的理解：一分钟时间戳差值60

#### 1.2 项目整体思路

##### 1.2.1 redis 有序集合

* 有序集合存储访问ip数据

有序集合中的元素是唯一的，而且可以采用score（本项目用时间戳作为排序分数）进行排序。可以保证访问ip不会重复，而且可以实时更新访问ip的score（因为存入相同元素的时候，虽然不会重复，但是score会随着当前时间更新）

##### 1.2.2  middleware中间件

* middleware实现接口检测

每次请求之前采用中间件对每个接口进行检测（但是值得注意的是，不检测redis存储的接口）

##### 1.2.3 celery定时任务

* celery定时任务实现删除过期元素，每60秒检测一次有无过期元素（和前一分钟的时间戳进行对比），有的话就删除（删除采用的是redis中的范围删除）

### 2 代码实现

#### 2.1 django端

##### 2.1.1 middlewares.py

```python
import redis
d = redis.Redis(host='127.0.0.1', port=6379, db=7, decode_responses=True)
class MemberOnlineMiddleWare(MiddlewareMixin):
    def process_request(self, request):
        path = request.path_info
        if path not in ('/user/online/'):
            ip = request.META['REMOTE_ADDR']
            import time
            d.zadd('onlines', {ip: int(time.time())})
```

##### 2.1.2 celery_task/celery_lk.py

* 值得注意的是，celery版本不能用4xx，要用5.0.0

```python
from celery import Celery
import os,  sys
CELERY_BASE_DIR = os.path.dirname(os.path.abspath(__file__))
print(sys.path)


# celery项目中的所有导包地址, 都是以CELERY_BASE_DIR为基准设定.
# 执行celery命令时, 也需要进入CELERY_BASE_DIR目录执行.
import django
sys.path.insert(0, os.path.join(CELERY_BASE_DIR, '../gogogo'))
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "gogogo.settings")

# 定义celery实例, 需要的参数, 1, 实例名, 2, 任务发布位置, 3, 结果保存位置
app = Celery('celery',
             broker='redis://127.0.0.1:6379/11',   # 任务存放的地方
             backend='redis://127.0.0.1:6379/12',   # 结果存放的地方
             include=['celery_task.tasks_beat'])
            # 由于上面配置了路径，所以导入时需要注意

app.conf.update(
   result_expires=3600,        #执行结果放到redis里，一个小时没人取就丢弃
)

from celery.schedules import crontab
# 配置定时任务
app.conf.beat_schedule = {
    'add-every-60-seconds': {
        'task': 'celery_task.tasks_beat.online_members',
        'schedule': 60.0,
        # 每60秒发送一次任务
    },
}


app.conf.timezone = 'Asia/Shanghai'

if __name__ == '__main__':
   app.start()
```

##### 2.1.3 celery_task/tasks_beat.py

```python
from __future__ import absolute_import, unicode_literals
# @app.task 指定将这个函数的执行交给celery异步执行

# 相当于装饰器，将下面函数打包给app
from celery_task.celery_lk import app

@app.task(bind=True)
def online_members(self):
    import redis, time
    d = redis.Redis(host='127.0.0.1', port=6379, db=7, decode_responses=True)
    after = int(time.time()) - 60
    d.zremrangebyscore('onlines', 0, after)
```

##### 2.1.4 views.py

```python
class OnlineView(APIView):
    def get(self, request):
        import redis, time
        d = redis.Redis(host='127.0.0.1', port=6379, db=7, decode_responses=True)
        num = d.zcard('onlines')
        return Response({'msg':'访问人数', 'num':num})
```

##### 2.1.5 urls.py

```python
from django.urls import path

from userapp import views


urlpatterns = [
    path('register/', views.RegisterView.as_view()),
    path('login/', views.LoginView.as_view()),
    path('dingding/', views.DingDingView.as_view()),
    path('test/', views.TestAPIView.as_view()),
    path('online/', views.OnlineView.as_view()),

]
```

##### 2.1.6 settings.py

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # mysql middleware
    # 'userapp.middlewares.VisitInfoMysqlMiddleWare',
    # redis middleware
    # 'userapp.middlewares.VisitInfoRedisMiddleWare',
    # mongo middleware
    # 'userapp.middlewares.VisitInfoMongoMiddleWare'，
    # userinfo middleware
    # 'userapp.middlewares.ChooseMiddleWare',
    # 'userapp.middlewares.ChooseMiddleWareB',
    # members online
    'userapp.middlewares.MemberOnlineMiddleWare',
    ]
```

### 2.2 Vue端

##### 2.2.1 headers.vue

```vue
<template>
        <div><a style="float:right" @click="outLogin">注销登录</a>
          <h3 style="margin-top:20px">           
          欢迎你
           <a-icon type="smile" theme="outlined" style="font-size:20px;color:blue"/>&ensp;
          {{username}}
            <a-icon type="fire" style="font-size:25px;color:blue"/>&ensp;
            实时人数：{{number}}
          </h3>         
        </div>
</template>
<script>
import { get_members_online } from '@/http/apis'
export default{
    data(){
        return{
          username:this.$route.query.username,
          number:''
        }
    },
    methods:{
      outLogin(){
        localStorage.removeItem('token')
        localStorage.removeItem('username')
        localStorage.removeItem('uid')
        alert('注销登录成功，稍后为您跳转登录页面')
        this.$router.push('/login')
      },
      getOnline(){
        get_members_online().then(res=>{
          console.log(res)
          this.number = res.num
        })
      }
    },
    created(){
      window.setInterval(this.getOnline, 5000)
    }
}
</script>

