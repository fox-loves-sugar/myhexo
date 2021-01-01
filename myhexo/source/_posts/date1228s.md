---
title: 优惠券
date: 2020-12-28 13:14:21
tags: [优惠券]
---

### 1 优惠券

* 充值以及充值成功之间要有优惠券

#### 1.1 生成随机码

```python
import random
import string

code = string.digits + string.ascii_letters
print(code)
# 生成字符串（4位）
def getCode():
    return ''.join(random.sample(code, 4))
						   # 样本
print(getCode())
# 重复率高
# 为了降低重复率
# 可以进行分位

def key(group):
    return '-'.join([getCode() for i in range(group)])
	# 列表推导式弊端，可读性降低，左边是逻辑，右边是循环
print(key(4))
```

#### 1.2 Django端

##### 1.2.1 models.py

```python
from django.db import models
from userapp.models import UserModel


class WalletModel(models.Model):
    money = models.DecimalField(max_digits=10, decimal_places=2)
    user = models.ForeignKey(UserModel, on_delete=models.CASCADE)

    class Meta:
        db_table = 'wallet_model'


class CouponModel(models.Model):
    coupon_choice = (
        ('1', '满300减50'),
        ('2', '八折优惠'),
        ('3', '无门槛10元红包')
    )
    user = models.ForeignKey(UserModel, on_delete=models.CASCADE)
    coupon = models.CharField(choices=coupon_choice, max_length=30)
    code = models.CharField(max_length=30)
    create_time = models.DateTimeField(auto_now_add=True)
    # 这里添加创建时间，是为了计算优惠券的有效时间

    class Meta:
        db_table = 'coupon_model'
```

##### 1.2.2 views.py

```python
class WalletView(APIView):
    def post(self, request):
        user_info = decodeToken(request)
        user_id = user_info.get('user_id')
        user = WalletModel.objects.filter(user_id=user_id).first()
        try:
            money = float(request.data.get('money'))
        except Exception as e:
            return Response({'msg': '充值错误', 'code': 400, 'error': e})
        if user:
               from decimal import Decimal
               # decimal 和 float 不能叠加，要把 float 转换
               user.money += Decimal(money)
               user.save()
        else:
            WalletModel.objects.create(money=money, user_id=user_id)

        if money < 100:
            return Response({'msg': '充值成功', 'code': 200})
        else:
            coupon_code = create_code(4)
            if 100 <= money < 300:
                CouponModel.objects.create(code=coupon_code, coupon='3', user_id=user_id)
                return Response({'msg': '充值成功，同时您获取了10元无门槛红包哦', 'code': 200})
            elif 300 <= money < 500:
                CouponModel.objects.create(code=coupon_code, coupon='1', user_id=user_id)
                return Response({'msg': '充值成功，同时您获取了一张满300减50的优惠券', 'code': 200})
            elif 500 <= money < 1000:
                CouponModel.objects.create(code=coupon_code, coupon='2', user_id=user_id)
                return Response({'msg': '充值成功，同时您获取了终生八折优惠', 'code': 200})
```

##### 1.2.3 utils.py

```python
# 生成优惠券随机码

import random
import string

code = string.digits + string.ascii_letters

def get_code():
    return ''.join(random.sample(code, 4))

def create_code(group):
    return '-'.join([get_code() for i in range(group)])
```

##### 1.2.4  MyAuthorization.py

```python
from django.conf import settings
from django.http import HttpResponse
from rest_framework_jwt.utils import jwt_decode_handler
import jwt
from jwt import exceptions


def decodeToken(request):
    token = request.META.get('HTTP_AUTHORIZATION')
    # print('this is token', token)
    # user_info = jwt_decode_handler(token[4:])
    # return user_info
    try:
        user_info = jwt.decode(token[4:], settings.SECRET_KEY)
        # print('this is user_info', user_info)
    except exceptions.ExpiredSignatureError:
        return HttpResponse('token已经失效')
    except jwt.DecodeError:
        return HttpResponse('token认证失败')
    except jwt.InvalidIssuer:
        return HttpResponse('非法的token')
    return user_info
```

#### 1.3 Vue端

```vue
<template>
    <div>
        <h1>钱包充值</h1>
        充值金额：<a-input v-model="costMoney" style="width:200px;"></a-input>
        <a-button @click="payWallet">充值</a-button>
    </div>
</template>

<script>
import axios from 'axios'
import { post_wallet } from '@/http/apis'
export default {
    data() {
        return {
            costMoney:'',
        }
    },
    methods: {
        payWallet(){
            let data={
                'money': this.costMoney
            }
            post_wallet(data).then(res=>{
                console.log(res)
            })
        }
}
</script>

<style scoped>

</style>
```
