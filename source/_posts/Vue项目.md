---
title: Vue项目
date: 2021-06-28 21:20:55
updated: 2021-07-05 10:18:46
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
        <!-- 按钮 -->
        <el-form-item class="btn">
          <el-button type="primary" @click="login()">登录</el-button>
          <el-button type="info" @click="resetLoginForm()">清空</el-button>
        </el-form-item>
</el-form>
```

1. label-width是输入框前的文字宽度。
2. 注意表单项el-form-item的prop应与数据名完全相同。
3. 通过ref标注DOM元素，用于拿到该表单实例，便于对整个表单进行校验：如果登录成功，则跳转到主页；否则提示登录失败。
4. :model用于绑定数据。
5. :rules用于表单的验证规则，比如用户名和密码的合法性。

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

**记住密码**

挂载时读取cookie

```javascript
mounted(){
    this.getCookie();
  }
```

```javascript
	//设置cookie
    setCookie(c_name, c_pwd, exdays) {
        let exdate = new Date(); //获取时间
        exdate.setTime(exdate.getTime() + 24 * 60 * 60 * 1000 * exdays); //保存的天数
        //字符串拼接cookie
        window.document.cookie =
            "username" + "=" + c_name + ";path=/;expires=" + exdate.toGMTString();
        window.document.cookie =
            "password" + "=" + c_pwd + ";path=/;expires=" + exdate.toGMTString();
    },
    // 读取cookie
    getCookie: function() {
        let that = this;
        if (document.cookie.length > 0) {
            let arr = document.cookie.split("; "); 
            for (let i = 0; i < arr.length; i++) {
                let arr2 = arr[i].split("="); 
                //判断查找相对应的值
                if (arr2[0] == "username") {
                    that.loginForm.username = arr2[1]; 
                } else if (arr2[0] == "password") {
                    that.loginForm.password = arr2[1];
                }
            }
        }
    },
    //清除cookie
    clearCookie: function() {
        this.setCookie("", "", -1); //修改两个值都为空，天数为-1
    }
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
- 主要区域：完成系统功能的操作区，本文统称为操作区域。

## 顶栏

采用flex布局，放置系统logo，系统名字，用户的虚拟头像以及下拉菜单（可以操作修改密码和退出登录）

### 下拉菜单

```vue
<!-- handleCommand用于触发事件 -->
<el-dropdown @command="handleCommand">
            <span class="el-dropdown-link">
              <i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="resetPassword">修改密码</el-dropdown-item>
              <el-dropdown-item command="logout">退出登录</el-dropdown-item>
            </el-dropdown-menu> 
  </el-dropdown>
```

### 修改密码

在操作区域显示修改密码组件内容，即修改密码表单。

```vue
<el-form ref="resetPasswordRef" :model="resetPasswordForm"  :rules="resetPasswordRules" label-width="80px">
      <el-form-item label="原密码" prop="oldPassword">
        <el-input v-model="resetPasswordForm.oldPassword" type="password"></el-input>
      </el-form-item>
      <el-form-item label="新密码" prop="newPassword">
        <el-input v-model="resetPasswordForm.newPassword" type="password"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="newPasswordConfirm">
        <el-input v-model="resetPasswordForm.newPasswordConfirm" type="password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="resetConfirm">确认修改</el-button>
        <el-button @click="resetFields">重置</el-button>
      </el-form-item>
  </el-form>
```

### 退出登录

只需要销毁token，这样再次请求时就不会携带token。销毁后返回登录页。

```javascript
window.sessionStorage.clear();
this.$router.push('/login');
```

## 侧边栏

设置导航栏，采用v-for循环遍历导入的数据，获取菜单栏子项，实现只允许展开一个子菜单，以及水平折叠收起菜单的功能，并为导航项配置路由。

```vue
<el-aside :width="isCollapse ? '65px' : '200px'">
    		<!--default-active将当前活跃的菜单高亮  -->
            <el-menu :default-active="$route.path"  background-color="#20588a" text-color="#fff" active-text-color="#ffd04b" 
            :unique-opened="true" :collapse="isCollapse" :collapse-transition="false" router>
              <el-submenu :index="indexf + ' '" v-for="(item, indexf) in menuList" :key="indexf">
                <template slot="title">
                  <i :class="item.icon"></i>
                  <span>{{ item.title }}</span>
                </template>
                  <el-menu-item :index="indexf + indexChild + ' ' " v-for="(itemChild, indexChild) in item.children" :key="indexChild">
                    <template slot="title">
                      <span>{{ itemChild.title }}</span>
                     </template>
                  </el-menu-item>
              </el-submenu>
              <el-menu-item :index="'/'+'view_counsellor'">
                <i class="el-icon-user"></i>
                <span slot="title">用户管理</span>
              </el-menu-item>
          </el-menu>
     </el-aside>
```

水平折叠收起菜单，动态绑定宽度

```html
<div class="toggle-button" :style="{width:(isCollapse ? '65px' : '200px')}" @click="toggleCollapse()"> ||| </div>
```

## 操作区域

### 超级管理员

#### 查看辅导员

**头部**：实现查找的输入框和添加辅导员的按钮。

```vue
<el-row :gutter="20">
    <el-col :span="12">
        <el-input placeholder="请输入内容" v-model="queryInfo.username" clearable @clear="getUserList()">
            <el-button slot="append" icon="el-icon-search" @click="search()" ></el-button>
        </el-input>
    </el-col>
    <el-col :span='12'>
        <el-button type="primary" @click="addDialogVisible=true">添加辅导员</el-button>
    </el-col>
 </el-row>
```

**中间**：显示辅导员信息的表格

```vue
<!-- slice实现前端分页，border表示带边框，stripe表示斑马纹，易于区分不同行 -->
<el-table :data="counsellorList.slice((currentPage-1)*pageSize,currentPage*pageSize)" border stripe>
    <!-- type表示添加索引 -->
    <el-table-column label="索引列" type="index" width="100px"></el-table-column>
    <el-table-column label="序号" prop="id"></el-table-column>
    <el-table-column label="用户名" prop="username"></el-table-column>
    <el-table-column label="密码" prop="password"></el-table-column>
    <el-table-column label="年级" prop="grades"></el-table-column>
    <el-table-column label="操作">
        <template>
        <!-- tooltip为按钮提示信息，enterable表示鼠标可以进入到tooltip中 -->
        <el-tooltip content="修改" placement="top-start" :enterable="false"><el-button type="primary" icon="el-icon-edit"></el-button></el-tooltip>
        <el-tooltip content="删除" placement="top" :enterable="false"><el-button type="danger" icon="el-icon-delete"></el-button></el-tooltip>
        <el-tooltip content="分配权限" placement="top-end" :enterable="false"><el-button type="warning" icon="el-icon-setting"></el-button></el-tooltip>
        </template>
    </el-table-column>
</el-table>
```

**底部**：分页功能

```vue
<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" 
   :page-sizes="[1,   2, 5, 10]" :current-page.sync="currentPage" :page-size.sync="pageSize"            layout="total, sizes, prev, pager, next, jumper"  :total="number">
</el-pagination>
```
