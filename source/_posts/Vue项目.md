---
title: Vue项目
date: 2021-06-28 21:20:55
updated: 2021-06-28 21:20:55
tags: 前端
---

## 准备工作

1. 安装vue 2.x

```
npm install vue@2.5.21 --save
```

--save 表示运行时依赖

--save-dev 表示开发时依赖，也就是项目部署后不需要的模块

2. 安装webpack

```
npm install wepack@3.6.0
```

3. 安装element ui

```
npm install element-ui
```

 	方法一：安装后在main.js引入，注意css需要单独引入

```
import Element from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
```

​	方法二：CLI3可以使用插件，进入项目命令行

```
vue add element
```

## 登录界面

登录流程：

1. 登录页面输入用户名和密码
2. 调用后台接口进行验证
3. 通过验证后，根据后台的相应状态跳转到对应项目页面

知识点：

1. http是无状态协议：服务器不知道客户端的状态，不会记录用户的信息

如果需要记录用户登录状态

1. 通过cookie在客户端记录状态：在客户端保持登录信息到下次与服务器的会话。

2. 通过session在服务器端记录状态：在服务器端保持会话信息

   如果出现跨域问题：网络协议或域名或端口有一个不同

3. 通过token方式维持状态，token由服务器生成返回给客户端

   ① 登录页输入用户名和密码

   ② 服务器验证通过之后生成该用户的token并返回

   ③ 客户端存储该token

   ④ 再次发送请求时都携带该token

   ⑤ 服务器验证token是否通过

### 登录表单

```vue
<el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user">
          </el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" prefix-icon="iconfont icon-3702mima">
          </el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btn">
          <el-button type="primary" @click="login()">登录</el-button>
          <el-button type="info" @click="resetLoginForm()">清空</el-button>
        </el-form-item>
</el-form>
```

1. 通过ref标注DOM元素，用于拿到该表单实例，便于对整个表单进行校验：如果登录成功，则跳转到主页；否则提示登录失败。
2. :model用于绑定数据。
3. :rules用于表单的验证规则，比如用户名和密码的合法性。

**校验表单**

```javascript
// 通过$refs获取标记ref属性的DOM元素
// 不传入回调函数，返回一个promise，采用async和await简化promise操作
this.$refs.loginFormRef.validate(async valid => {
    // 如果校验为false
    if(!valid) return;
    // 发送axios请求，同时解构data属性
    const {data: res} = await this.$http.post('login', this.loginForm);
    // 判断状态码
    if(res.meta.status !== 200) return this.$message.error('登录失败!');
    this.$message.success('登录成功!');
	// 将登录成功后的token保存在客户端的sessionStorge中
    window.sessionStorage.setItem("token", res.data.token);
    // 登录成功后跳转到主页
    this.$router.push('/home');
}
```

**表单验证规则**

```javascript
username: [{ required: true, message: '请输入账号', trigger: 'blur' }, {min: 5, max: 8,message: "长度为5到8位", trigger: "blur"}], 
password: [{ required: true, message: '请输入密码', trigger: 'blur' }, {min: 6,max: 15,message: "长度为6到15位", trigger: "blur"}]
```

## 路由导航守卫

通过路由导航守卫控制访问权限，如果用户没有登录，不可以访问指定页面，应该重新返回到登录页。

```javascript
router.beforeEach((to, from, next) => {
  // 如果访问登录页，可以next
  if(to.path === '/login') return next();
  // 从sessionStorge获取保存的token
  const tokenStr = window.sessionStorage.getItem('token');
  // 如果没有token，则跳转到登录页
  if(!tokenStr) return next('/login');
  // 如果有，可以继续
  next();
})
```

## 主页

### 主页布局

首先确定布局容器，本项目选用顶栏+侧边栏+主要区域。

- 顶栏：logo，系统名字，用户的虚拟头像，以及关于修改密码和退出登录的操作。
- 侧边栏：导航菜单。
- 主要区域：完成系统功能的操作区。

### 顶栏

采用flex布局，放置系统logo，系统名字，用户的虚拟头像以及下拉菜单（可以操作修改密码和退出登录）

#### 修改密码



#### 退出登录

只需要销毁token，这样再次请求时就不会携带token。销毁后返回登录页。

```javascript
window.sessionStorage.clear();
this.$router.push('/login');
```

## 

