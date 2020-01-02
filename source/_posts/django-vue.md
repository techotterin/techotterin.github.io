---
title: 使用 Django 和 Vue 构建 Web 开发框架
date: 2020-01-03 13:32:11
author: techotter
tags:
  - python
  - vue
  - django
categories:
  - 后端开发
---

## 准备环境

为了开始我们的项目，我们需要确保以下组件已经安装在基于 Debian 9 的系统中：

- Django >= 1.11
- Python >= 3.6
- MySQL >= 5.7
- Node.js >= 10.15
- vue-cli >= 2.0

<!-- more -->

## 安装步骤

### 安装 Node.js

```bash
wget https://nodejs.org/dist/v10.16.3/node-v10.16.3-linux-x64.tar.xz
xz -d node-v10.16.3-linux-x64.tar.xz
tar xvf node-v10.16.3-linux-x64.tar
mv node-v10.16.3-linux-x64 /usr/local/node-v10.16.3
ln -sf /usr/local/node-v10.16.3/bin/node /usr/local/bin/
ln -sf /usr/local/node-v10.16.3/bin/npm /usr/local/bin/
node --version
npm --version
```

### 安装 Python 和 pip

```bash
wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tar.xz
xz -d Python-3.7.4.tar.xz
tar xvf Python-3.7.4.tar
cd Python-3.7.4
./configure
make && make install
python3 -V
curl https://bootstrap.pypa.io/get-pip.py | sudo python3
pip --version
```

### 安装 Django

```bash
pip install django==1.11.13
```

### 安装 vue-cli

```bash
npm install -g @vue/cli
vue --version
```

### 安装 MySQL

下载并安装 MySQL Server，参考 [MySQL Archives](https://downloads.mysql.com/archives/community/). 安装完成后，安装 MySQL Python 客户端：

```bash
pip3 install mysqlclient
```

## 构建后端

### 创建 Django 项目

```bash
django-admin startproject myproject
cd myproject
python3 manage.py startapp backend
```

### 配置数据库

在 `myproject/settings.py` 中配置 MySQL 数据库：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}
INSTALLED_APPS += ['backend']
TIME_ZONE = 'Asia/Shanghai'
```

### 创建模型

在 `backend/models.py` 中定义一个简单的模型：

```python
from django.db import models

class ChinaCities(models.Model):
    city_name = models.CharField(max_length=255)
    city_code = models.IntegerField(default=0)
    create_time = models.DateTimeField(auto_now_add=True)
```

### 创建视图

在 `backend/views.py` 中添加视图来处理城市数据：

```python
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from .models import ChinaCities
import json

@require_http_methods(["GET"])
def add_city(request):
    response = {
        'message': 'success',
        'code': 100
    }
    try:
        city = ChinaCities(city_name=request.GET.get('city_name'), city_code=request.GET.get('city_code'))
        city.save()
    except Exception as e:
        response['message'] = str(e)
        response['code'] = 99
    return JsonResponse(response)

@require_http_methods(["GET"])
def city_list(request):
    response = {
        'message': 'success',
        'code': 100
    }
    try:
        cities = ChinaCities.objects.all()
        response['data'] = json.loads(serializers.serialize("json", cities))
    except Exception as e:
        response['message'] = str(e)
        response['code'] = 99
    return JsonResponse(response)
```

### 配置 URL 路由

在 `backend/urls.py` 中添加 URL 路由：

```python
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'add_city$', views.add_city),
    url(r'city_list$', views.city_list),
]
```

在 `myproject/urls.py` 中包含这些路由：

```python
from django.conf.urls import include, url
from django.contrib import admin
from backend.urls import urlpatterns as backend_urls

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'api/', include(backend_urls)),
]
```

## 构建前端

### 创建 Vue.js 项目

```bash
vue create frontend
```

### 配置和运行

进入 `frontend` 目录，启动 Vue.js 应用：

```bash
npm install
npm run serve
```

至此，后端和前端的基本架构已经搭建完毕，可以开始进一步开发和测试了。

```markdown
### 安装依赖包

1. 安装 `element-ui` 框架和 `vue-resource` (HTTP 请求等相关包):

   ```bash
   cd frontend
   npm install
   npm install element-ui
   npm install vue-resource
   ```

### 配置 Vue 加载组件

在 `src/main.js` 下添加以下配置:

```javascript
import VueResource from "vue-resource";
import ElementUI from "element-ui";
import "element-ui/lib/theme-chalk/index.css";

Vue.use(VueResource);
Vue.use(ElementUI);
```

### 创建 Vue 组件

在 `src/components` 文件夹下新建一个组件 `Home.vue`:

```vue
<template>
  <div class="home">
    <el-row display="margin-top:10px">
      <el-col :span="4"><el-input v-model="name" placeholder="请输入城市名称"></el-input></el-col>
      <el-col :span="4"><el-input v-model="code" placeholder="请输入城市代码"></el-input></el-col>
      <el-button type="primary" @click="addCity(name, code)">新增城市</el-button>
    </el-row>
    <el-row>
      <el-table :data="cityList" style="width: 100%" border>
        <el-table-column prop="id" label="编号" min-width="100">
          <template scope="scope">{{ scope.row.pk }}</template>
        </el-table-column>
        <el-table-column prop="city_name" label="城市名称" min-width="100">
          <template scope="scope">{{ scope.row.fields.city_name }}</template>
        </el-table-column>
        <el-table-column prop="city_code" label="城市代码" min-width="100">
          <template scope="scope">{{ scope.row.fields.city_code }}</template>
        </el-table-column>
        <el-table-column prop="create_time" label="创建时间" min-width="100">
          <template scope="scope">{{ scope.row.fields.create_time }}</template>
        </el-table-column>
      </el-table>
    </el-row>
  </div>
</template>

<script>
export default {
  name: 'home',
  data() {
    return {
      name: "",
      code: 0,
      cityList: [],
    }
  },
  methods: {
    fetchList() {
      this.$http.get("/api/city_list").then(res => {
        if (res.body.code == 100) {
          this.$message({
            type: "success",
            showClose: true,
            message: "加载成功",
          });
          this.cityList = res.body.data;
        } else {
          this.$message({
            type: "error",
            showClose: true,
            message: "加载失败",
          });
        }
      })
    },
    addCity(name, code) {
      let options = {
        params: {
          "city_name": name,
          "city_code": code
        },
        emulateJSON: true,
      };
      this.$http.get("/api/add_city", options).then(res => {
        if (res.body.code == 100) {
          this.$message({
            type: "success",
            showClose: true,
            message: "添加成功",
          });
          this.fetchList();
        } else {
          this.$message({
            type: "error",
            showClose: true,
            message: "添加失败",
          });
        }
      })
    }
  },
  mounted() {
    this.fetchList();
  }
}
</script>
```

### 添加路由配置

1. 在 `src/router.js` 下新增路由配置:

   ```javascript
   import Home from './views/Home.vue';
   import Router from 'vue-router';

   export default new Router({
     mode: 'history',
     base: process.env.BASE_URL,
     routes: [
       {
         path: '/',
         name: 'home',
         component: Home
       },
     ]
   });
   ```

### 构建和运行项目

1. 在 `frontend` 目录下运行 `npm run dev` 启动 Node 服务器，在浏览器中打开地址:
   
   ```bash
   npm run dev
   ```

2. 使用以下命令构建打包所有资源到 `dist` 目录下:

   ```bash
   npm run build
   ```

### 整合前后端

1. 将 Django 的 `TemplateView` 指向我们刚才生成的前端 `dist` 文件:

   ```python
   from django.urls import path
   from django.views.generic.base import TemplateView

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', TemplateView.as_view(template_name="index.html")),  # 添加这行
   ]
   ```

2. 在 `myproject` 目录的 `settings.py` 下配置静态文件路径:

   ```python
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
           'DIRS': ['frontend/dist'],  # 修改这句
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

   STATICFILES_DIRS = [
       os.path.join(BASE_DIR, "frontend/dist/static/"),
   ]

   STATIC_URL = '/static/'
   ```

3. 运行项目查看是否正常:

   ```bash
   python3 manage.py runserver 8080
   ```
