---
title: 尚品甄选项目笔记
layout: default
parent: 尚品甄选
---



# 11月20日笔记

### Maven

* Maven的环境：Maven的本地仓库是在settings文件里面配置，项目对应的Maven和Maven对应的本地仓库都是独立的。不同项目使用的Maven不同。
  TODO: 不同项目使用不同的Maven如何配置？
```xml
<localRepository>xxx</localRepository>
```

* 在Idea中，自带Maven，所以我使用的Maven就是自带的Maven。

mac使用homebrew安装Maven（但是好像只有最新版本的？）







---

### Git的dev分支和工作流程

* 开发的git使用：本地创建dev分支，不与origin/master进行同步，这个分支只进行开发。当要提交代码时，origin/master和本地master进行同步，然后dev merge 本地master，并进行master的上传。
* 然后再次dev开发的时候，要保证版本的一致。这里直接把本地master merge到dev就行。因为master刚刚版本控制过（master=dev+其他），这里一定不会出现冲突。

问题：idea git log图看不懂！



---

### Docker 迁移






```shell
df 
# 查看存储空间
```

* Docker 迁移操作
* 把当前的一个Docker容器压缩成镜像，然后移动到另一个服务器的Docker中运行出来



```shell
docker commit 容器名称/容器的id 镜像名称			  # 把docker容器保存成一个镜像
docker save -o 镜像tar文件名称 镜像名称/镜像id		 # 把镜像保存为tar文件
docker load -i 镜像名称							  # 把tar文件恢复成为一个镜像
```

示例代码

```shell
docker commit redis01 myredis				     # 将redis01容器保存为一个镜像
docker save -o myredis.tar myredis 				 # 将myredis镜像保存为一个tar文件
docker rmi myredis								 # 删除之前的myredis镜像
docker load -i myredis.tar 						 # 将myredis.tar恢复成一个镜像
```

==测试了一下，出现的问题是，由于架构不一样导致==



==如果MySQL容器使用了镜像卷，那压缩成镜像的过程会出问题吗？==

（测试完了，好像镜像卷的内容确实不会保存进来）

==查看一个容器是否是用了数据卷==



---

### ubuntu清理磁盘空间

```
root@ubuntu:/home/jasper# docker load -i mydb.tar
ad6b69b54919: Loading layer [==================================================>]  72.55MB/72.55MB
fba7b131c5c3: Loading layer [==================================================>]  338.4kB/338.4kB
0798f2528e83: Loading layer [==================================================>]  9.556MB/9.556MB
a0c2a050fee2: Loading layer [==================================================>]  4.202MB/4.202MB
d7a777f6c3a4: Loading layer [==================================================>]  2.048kB/2.048kB
0d17fee8db40: Loading layer [==================================================>]  53.77MB/53.77MB
aad27784b762: Loading layer [==================================================>]  5.632kB/5.632kB
9b64bb048d04: Loading layer [==================================================>]  3.584kB/3.584kB
35ba198e64f5: Loading layer [============================================>      ]  279.6MB/313.2MB
write /usr/bin/myisamchk: no space left on device

```



```
root@ubuntu:/home/jasper# df -h
Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.9G     0  1.9G   0% /dev
tmpfs                              392M  1.4M  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  9.2G   85M 100% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/loop1                          92M   92M     0 100% /snap/lxd/29631
/dev/nvme0n1p2                     1.8G  206M  1.5G  13% /boot
/dev/nvme0n1p1                     952M  6.4M  945M   1% /boot/efi
/dev/loop0                          60M   60M     0 100% /snap/core20/2383
/dev/loop3                          62M   62M     0 100% /snap/lxd/22761
tmpfs                              392M     0  392M   0% /run/user/1000
/dev/loop5                          60M   60M     0 100% /snap/core20/2437
/dev/loop4                          34M   34M     0 100% /snap/snapd/21761
overlay                            9.8G  9.2G   85M 100% /var/lib/docker/overlay2/e95fc76e4cb888b577e614a1183e5a1f612fe7cfa3af4c54d7aafd42981e1483/merged
overlay                            9.8G  9.2G   85M 100% /var/lib/docker/overlay2/0e8475bc1f45506a754187cb9bc4a1e0d841316703974d02749d14202ac82900/merged

```

真的没有想到，我的ubuntu20虚拟机磁盘空间竟然用完了！



---

### Ubuntu虚拟机扩容



```
root@ubuntu:/home/jasper# sudo fdisk /dev/nvme0n1

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

GPT PMBR size mismatch (41943039 != 83886079) will be corrected by write.

Command (m for help): p

Disk /dev/nvme0n1: 40 GiB, 42949672960 bytes, 83886080 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2495C3D7-088B-43B8-98FB-C283FBF89039

Device           Start      End  Sectors  Size Type
/dev/nvme0n1p1    2048  1953791  1951744  953M EFI System
/dev/nvme0n1p2 1953792  5668863  3715072  1.8G Linux filesystem
/dev/nvme0n1p3 5668864 41940991 36272128 17.3G Linux filesystem

Command (m for help): d
Partition number (1-3, default 3): 3

Partition 3 has been deleted.

Command (m for help): n
Partition number (3-128, default 3): 
First sector (5668864-83886046, default 5668864): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (5668864-83886046, default 83886046): 

Created a new partition 3 of type 'Linux filesystem' and of size 37.3 GiB.
Partition #3 contains a LVM2_member signature.

Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): w

The partition table has been altered.
Syncing disks.


```





---

### 制作 SpringBoot Docker 镜像



> 初始项目在maven导入依赖的时候总是报错，报错的原因是3.0.5版本的spring-boot-starter-parent似乎找不到。
>
> 不太清楚原因。我考虑下载一个单独的maven，不用idea自带的了。（自带的太不方便了，mvn指令也执行不了）
>
> 我换了maven库存在的3.2.3试了一下，同样报错：
>
> ```
> Unresolved plugin: 'org.springframework.boot:spring-boot-maven-plugin:3.0.5'
> ```
>
> **查找设置发现maven的路径竟然是错的，还有C盘！这个东西也能记录到项目中吗？**
>
> 然后我把maven的路径修改成正确的（idea自带的），然后就好使了。。。



参考博客：

* [centos7安装docker](https://cloud.tencent.com/developer/article/1701451)
* [ubuntu扩容](https://blog.csdn.net/qq_44339029/article/details/120597169)
* [ubuntu扩容](https://blog.csdn.net/qq_34160841/article/details/113058756)



# 11月21日笔记

## 前端项目

mock是模拟数据的意思，模拟后端给前端发送过来的数据

<img src="/Users/jmjin/Desktop/Snipaste_2024-11-21_14-03-25.png" alt="Snipaste_2024-11-21_14-03-25" style="zoom:50%;" />

* Local storage本地存储是干什么的？好像是用来前端登录记录用的。他和cookie有啥区别
* 需要下载和安装vue3？我好像没下载。。。
* TODO：弄清楚登录过程index.vue的相关代码，和登录过程的调用
* 啥是路由前置守卫？



userinfo不是通过登录的vue调用的，而是其他的组件调用的。

分析过程：在f12检查中发现了userinfo的request，url是/api/userinfo，然后api中找到/api/userinfo这个请求是getUserinfo的。然后搜索是谁调用了getUserinfo，找到了是permission.js。permission.js是路由的前置守卫，应该是入口文件main.js加载的。

vscode快捷键设置：

Code->首选项->键盘快捷方式

修改go back和go forward



### main.js

router

pinia



index.vue是干啥的



### router

**配置和管理前端路由的核心文件，它使用了 Vue Router 库来创建和控制页面路由**

* 当你在 JavaScript 或 TypeScript 项目中使用 `import router from './router'` 时，系统会自动查找 `router` 文件夹下的 `index.js` 或 `index.ts` 文件。这是因为 `index.js`（或 `index.ts`）被视为文件夹的默认入口文件。
* 如果 `import redirect from './modules/redirect'` 执行时，系统首先会查找一个名为 `redirect.js`（或 `.ts` 等）的文件。如果没有找到这样的文件，它才会查找名为 `redirect` 的目录，并尝试加载该目录下的 `index.js` 文件。
* `[...home]` 使用的是 JavaScript 的**扩展运算符**（spread operator），它用于将一个数组的所有元素展开到另一个数组中。在这个语境中，`home` 可能是一个包含多个路由对象的数组，使用扩展运算符可以将这些路由对象直接展开并合并到新的数组中，这种方式常用于组合和管理路由配置。
* **路由数组** 是一个数组，其中每个元素都是一个**路由对象**，定义了路径、组件和其他路由元数据。在 Vue Router 中，这个数组用于配置路由器实例，告诉 Vue 应用如何根据不同的 URL 地址加载不同的组件。
* `const Home = () => import('@/views/home/index.vue')` 的语法：这是一个**动态导入**的例子，也是一个利用箭头函数返回一个 Promise 对象的表达式，这个 Promise 将异步地解析为一个 Vue 组件。这种语法是实现路由懒加载的常见方式，在 Vue Router 中广泛使用。当路由被访问时，对应的组件才会被加载，有助于减少应用的初始加载时间。
* 编程式导航：指的是使用 JavaScript 代码（而非链接点击）来实现页面跳转。在 Vue Router 中，你可以使用 `router.push` 或 `router.replace` 方法来编程式地导航到不同的路由。





### promise对象

**Promise** 是 JavaScript 中用于处理异步操作的对象。它表示一个尚未完成但预计将来会完成的操作的结果。

```javascript
let myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Data received");
  }, 3000);
});

myPromise.then(data => {
  console.log(data);  // 输出: Data received
});

```



### 路由

[vue路由测试](https://play.vuejs.org/#eNqFVVtv2zYU/itn6gArmC05btEHTXXTFcWyYZeiLfYy7UGWji02EsmRlOPA8H/fIambnaRD4Fg61++c7yN9DJqc8eirDpKANVIoA0coFOYG30kJJ9gq0cBs3+Is412AEq1B1Xmi2L+ObpvX+3IpI5+b8aFqSJ+rjANErcbQp/v3RrTchLMXlDa7CuZBl07YUoONrCl/bQPT6np9i3UtbLPv0phenVm6L3rQRgm+W79vlULeIQaZmypJ484HxyN87xzRtq3rj+SE08mViX2dlOf7vuAnh/I3xu/AiDdZEGfB+mdBz3ArGkzj0f9sRr4hy5D2zr49ykvjvmdqeTmv9RfDe4i7uM6dxsNiaF9+l0+y+Ts2Qj3cMm3oa94Zfd0py4uBzYFPO6Br3ZPaGzpme9rtQGdxg2WUgOC6Y0PDG/jbjnL0vMAsnhEsQcU4UZaMbU/z8zC3x/PYsbcN/ueilaJW03nDoy1Y+VUkT+0nvHI9PVB6PJE8M44HN2iJ27yt+9q09ek+rFR1oZg0RM5FgmvboKlEqRP/BrATX4SDH171JgBD4CIvThXJVldhP7Y7J9DtxP4nxZKk+470cnFQVuseHh2TlTduWmMEh5uiZsUdSXPAcKlOH/hIZmfEjhODRtPaozNKjyiiGcqn75Ej0Pl3lMyHp2fFeMHnEB/SRia+ict6ep/GXBWV1UGHyGtgh5O1K0KvuC8T/duieoi6tLdvYUYg+rXTmKH3jLmeKoW0owLDI7h8IrnvfAKrIargxfQ/lA0LHjmr8w3W3X3w2dVMIGWchoH9ohEl1pFRrCE2fccsgCY/1Mh3piLjaknc+pujr3TOqedk0eSSrg/BiVU3WtY5dBYMks2CkRtrzoLKGKmTOG65vNtFtON4jLh5Fb2MlnFJJ2tijVA3i40S99rdV1ngNmtr31BQXOLeCFHrRS7Zcy0eBd68jl5H13HNNjFVjxkv8eBq94unMY0mQWzZ7mJIKwtWo/pTGkaCORs2p9+Z+1+dzagWB6BFhcXdE/av+uAhf1RI0+1xMpzJFWnOuz98/gMP9Dw4icW2puhvOD+hFnVrMfqwn1peEuxJnEP7i+OM8d0X/eFgkOt+KAt0FLIj8v03Rh/hvoxeTbaozUONOiq0/aGhX6w5aY1xn7cRqkSVwEoegMCyEl4sl8sf3d1H5RhfbATdKk0C10t5cHaZlyWBHSzUJeNUFtaQww/08Tenz65xSzf+NLJaTTuP5UcARVFMACSwpL9VVyE4/QesCg/V)

main.js

```javascript
import { createApp } from 'vue'
import router from './router'
import App from './app.vue'

createApp(App) // 创建一个新的 Vue 应用实例，将 App 作为根组件。
  .use(router) // 将 Vue Router 插件注册到这个应用中，这允许你在应用中使用路由功能。
  .mount('#app') // 将 Vue 应用挂载到 DOM 中的一个具体元素上（通常是一个 id 为 app 的 div 元素）
```

* 





# 11月22日笔记

## 前端知识复习



## 开发登录功能

* vo的含义：Value Object 值对象，用来传递数据

* DTO：data transfer object 数据传输对象

* `mybatis.mapper-locations=classpath:/mapper/*/*.xml`：mybatis的classpath是resources文件夹。

* ```xml
    <mapper namespace="com.jasper.manager.mapper.SysUserMapper">
        <select id="selectUserByName" parameterType="com.jasper.model.entity.system.SysUser">
            select
            <include refid="columns"></include>
            from sys_user where username = #{username}
        </select>
    </mapper>
    ```
    
    xml的配置文件：namespace是把mapper的java接口和xml配置文件做对应（写好这个就有mybatisx小鸟了！）。id是方法名。parameterType是返回类路径。`#{username}`中用#表示的是**方法的参数属性**名称。
    
* 是否需要@Mapper？如果你的项目中配置了 `@MapperScan`，就 **不需要** 在每个 `Mapper` 接口上加 `@Mapper` 注解。

* spzx-manager是否引入了spzx-model？manager引入了common-service依赖





**写一个mapper犯的错误：**

* `parameterType="java.lang.String" resultType="com.jasper.model.entity.system.SysUser"`注意一个是参数类型，一个是返回类型
* `username = #{userName}`注意#在大括号前面
* `#{userName}`这个是方法的传参的名字，如果是对象的话，这里直接写属性的性，不用写对象.属性





* interface用加public吗？
* mapper是class还是interface
* 为什么用postman接口测试的时候不会出现跨域问题？而前端工程会出现跨域问题？

* 跨域是保护谁的？老师说是保护被访问者（服务器）的安全，但是GPT说是保护访问者（用户）的隐私。

* 为啥前端发过来的请求数据到后端就是null了？



抽象类 给别人继承的类 工具类 





## 11月23日知识点预习

* Assert，把判断都改成断言
* redis
* 异常体系
* aop
* 注解（target是注解的范围，是类还是方法啥的）（retention是编译时有效还是运行时有效







# 11月23日笔记

## Java核心知识点





## 后端开发

### getUserInfo

* 写controller函数的时候返回result携带的data类型、传参类型应该**去api文档获取**。
* 传参的类型也分很多种，例如**Body参数、Header参数**。
    传递Body参数需要加@RequestBody（将请求体（Body）中的 JSON 或其他格式的数据解析为方法参数对应的对象）；
    header参数直接写在传参就行？（可以通过直接在方法参数中使用 `@RequestHeader` 注解来绑定）



HTTP 报文分为三个主要部分：**起始行**、**Header（头部）** 和 **Body（主体）**。

**没有Body的报文**

```
GET /api/resource HTTP/1.1
Host: example.com
Authorization: Bearer abc123
User-Agent: PostmanRuntime/7.32.0
Accept: */*

```

<font color=red>GET 请求通常没有 Body。</font>



**POST 请求（带 JSON Body）**

```
POST /api/createUser HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer abc123
Content-Length: 36

{
    "name": "John",
    "age": 30
}
```

注意一下第三行`Content-Type: application/json`

<font color=red>Body 部分：实际提交的数据内容，格式由 **`Content-Type`** 决定（此处是 JSON）。</font>



**表单提交（默认 Content-Type: `application/x-www-form-urlencoded`）**

```
POST /api/createUser HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 17

name=John&age=30
```



### menus







### 传递token



* 前后端分离，请求跨域时，请求能够携带cookie的信息吗？





### 断言、异常处理







### 验证码







# 11月24日-26日笔记

**前端代码分析**

```
```





* 什么时候需要扫描注释？要看应用和引入包的依赖关系

* 以前的框架options是屏蔽的，现在不处理不屏蔽





* 感觉系统存在一个bug，就是前端携带token，但是redis找不到token的时候，这时候前端会一直加载。这个应该让前端处理一下吧？



@RequestBody是接post请求的，get请求在url中的请求参数不用注释接，直接写在参数中





**关于字符串为空的验证**

```java
PowerAssert.isTrue(token == null || token.isEmpty(), "token不能为空");
```

这样写似乎不够正确，我实际上运行的时候也出了问题。





## 前端功能添加

### 1. 路由

暂时把system路由添加到全局里了

```javascript
export const fixedRoutes = [...home, ...system]
```

添加system路由

```javascript
const Layout = () => import('@/layout/index.vue')
const sysUser = () => import('@/views/system/sysUser.vue')
const sysRole = () => import('@/views/system/sysRole.vue')
const sysMenu = () => import('@/views/system/sysMenu.vue')

export default [
  {
    path: '/system', // 路由的URL路径
    name: 'system', // 路由名称，
    component: Layout, // 页面
    meta: {
      // 包含路由元信息的对象，可以包含任何你需要的数据，常用于存储权限信息、页面标题等
      title: '系统管理',
    },
    icon: 'Setting', // 小图标 https://element-plus.org/zh-CN/component/icon.html
    children: [
      // 子路由
      {
        path: '/sysUser',
        name: 'sysUser', // TODO path,name是匹配数据库的
        component: sysUser,
        meta: {
          title: '用户管理',
          affix: false,
        },
        icon: 'User',
      },
      {
        path: '/sysRole',
        name: 'sysRole',
        component: sysRole,
        meta: {
          title: '角色管理',
          affix: false,
        },
        icon: 'EditPen',
      },
      {
        path: '/sysMenu',
        name: 'sysMenu',
        component: sysMenu,
        meta: {
          title: '菜单管理',
          affix: false,
        },
        icon: 'Operation',
      },
    ],
  },
]
```

创建路由对应的vue页面







### 2. 页面





* 在Vue中，`$event` 是一个特殊变量，代表原生DOM事件的事件对象。这可以用来访问标准的事件属性，比如 `event.target`, `event.preventDefault()` 等。在你的代码中，`doRemove($event)`这样传递 `$event` 是为了在方法内部可能需要使用到这个事件对象，尽管在示例中并未直接使用它。
* 









# 11月27日笔记

开发sysRole、sysUser功能，查找；==增/改==；删除

* 作用域` #default="scope"` 
* 
* 为什么修改是put？添加是post？
* 添加和修改可以放到一个函数里面实现
* `<if test="roleName != null and roleName !=''">` 这到底是啥语言？既不是sql又不是java





* TODO axios传递token方法？
* 



```xml
<update id="updateSysRole">
    update sys_role
    <set>
        <if test="roleName != null and roleName !=''">
            role_name=#{roleName},
        </if>
        <if test="roleCode != null and roleCode != ''">
            role_code=#{roleCode},
        </if>
        <if test="description != null and description != ''">
            description=#{description},
        </if>
    </set>
    <where>
        id=#{id}
    </where>
</update>
```

忘加逗号了！



### 传参和注释大总结

`@PathVariable`：GET方法不能body传参（没有body），**路径变量**是URL `{var}` 中表示的参数

`@RequestHeader`：GET方法不能body传参（没有body），从HTTP请求头中获取值



`@RequestParam`：GET和POST请求都可以用。GET方法不能body传参（没有body），请求参数是 `?` 后面的KV对。如果用于POST获取传参，也要求**请求体Body部分是简单的KV对**。例如如下：

```
POST /submit
Content-Type: application/x-www-form-urlencoded

name=John
```



`@RequestBody`：用于POST或PUT请求，且**获取body部分比较复杂类型的参数，例如json**。**把请求体中的json数据反序列化为 Java 对象**。（要添加，否则前端传来的json参数无法解析）。一般作为Controller函数中，param是**类**的注释。

```
POST /create
Content-Type: application/json

{
    "name": "John",
    "age": 25
}
```

```java
@PostMapping("/create")
public String create(@RequestBody User user) {
    return "User: " + user.getName() + ", Age: " + user.getAge();
}
```





```java
@GetMapping("/getUserInfo")
public Result<SysUser> getUserInfo(@RequestHeader("token") String token) {
    // service可能需要token
}
```



**传递文件**

```java
public Result<String> fileUpload(@RequestParam(value = "file")MultipartFile multipartFile) {

}
```







`@ModelAttribute` ==这是干啥的？？？==





### async 和 then 两种写法

两种方式处理异步操作

```javascript
async() => {
    const {code} = await DeleteSysRoleById(row.id)
    if (code === 200) {
        ElMessage.success('删除成功')
        fetchData()
	} else {
        ElMessage.error('删除失败')
	}
}
```





```javascript
() => {
	DeleteSysRoleById(row.id).then(response => {
		if (response.code === 200) {
			ElMessage.success('删除成功')
			fetchData()
		} else {
			ElMessage.error('删除失败')
		}
	})
}
```





```javascript
const deleteById = row => {
  // NOTE then实现
  ElMessageBox.confirm('此操作将永久删除该记录, 是否继续?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  })
    .then(() => {
      DeleteSysRoleById(row.id).then(response => {
        if (response.code === 200) {
          ElMessage.success('删除成功')
          fetchData()
        } else {
          ElMessage.error('删除失败')
        }
      })
    })
    .catch(() => {
      ElMessage.info('取消删除')
    })

  // NOTE 这里是await用法
  // ElMessageBox.confirm('此操作将永久删除该记录, 是否继续?', '提示', {
  //   confirmButtonText: '确定',
  //   cancelButtonText: '取消',
  //   type: 'warning',
  // })
  //   .then(async () => {
  //     let response = await DeleteSysRoleById(row.id)
  //     if (response.code === 200) {
  //       ElMessage.success('删除成功')
  //       fetchData()
  //     } else {
  //       ElMessage.error('删除失败')
  //     }
  //   })
  //   .catch(() => {
  //     ElMessage.info('取消删除')
  //   })

}
```





* 在mybatis xml文件中，`<=` 和关键字冲突，所以要用 ` &lt;= `
* ==在sysUser插入的时候，如果username重名，会报错！这个为啥呢？==
* 









* `@SneakyThrows` 简化异常处理

* ```
    <el-upload
      class="avatar-uploader"
      action="http://localhost:8501/admin/system/fileUpload"
      :show-file-list="false"
      :on-success="handleAvatarSuccess" 
      :headers="headers"
    >
    ```
	传递on-success，用于图片回显
	
* 报错`There is no getter for property named 'avator' in 'class com.jasper.model.entity.system.SysUser`：原因是在sql的xml中写错了，把avatar写成了avator

* 有些好奇前端传递的Dto对象数据是如何封装到后端，反序列化成dto对象的。但是发现chrome好像没办法查看post的

* 如果post请求传递的数据是dto，前端js也必须有一个dto相同结构的对象，并且调用api传递的参数是这个对象。

* ```html
    <el-dialog v-model="dialogVisible" :title="dialogTitle" width="30%">
    ```
    **v-model是值的双向绑定，:是属性值绑定**
    
* 项目AOP，也就是LoginAuthInterceptor，会在每一个controller请求拦截，然后都会去redis检查一下token

* 前端，弹出的修改/增加框中显示数据，要把每个input的栏目和数据进行绑定！

* 为角色分配菜单使用的前端的树形结构







# 12月1日笔记

* （前端）resolve函数不太理解
* java读取excel的框架：easyexcel
* xml中`<scope>test</scope>`这个关键字是什么意思？干什么用的？老师说的是这个吗？
* java什么叫做单例里面不能有共享变量？为啥Listener是单例呢？
* SpringMVC整个都是单例模式，意思是里面的bean都是单例的。所以这种模式下，不可以在里面注入容器类型，否则多线程会出错。（所有的Bean都有这个限制，
* 构造器注入？无参构造？NoArgsConstructor
* 什么是BOM操作和DOM操作？用来存xlsx数据的？
* aixos默认返回的类型是outputstream

Spring Bean对象都是无状态的，无状态的类天然线程安全。

例如在Controller、Service等类中，只声明了方法，并没有值属性等表示状态的属性，天然是安全的。同时也要注意，在Spring的Bean中添加属性的时候，要格外注意==属性不能是有状态的==。通俗的话来讲，不能是用来存值的属性。不然，不同的线程调用单例，都会对值进行修改，这样就会出错！





## 线程、协程调度

###  1. Java 中线程调度的底层实现

#### **1.1 Java线程依赖操作系统线程**

Java 的线程调度和管理是依赖操作系统的实现：

- **基于原生线程（Native Threads）：**
    Java 的线程是操作系统线程（如 Linux 的 pthread 或 Windows 的线程）的封装，依赖操作系统内核来调度。
- 使用 JVM 调用操作系统的线程 API：
    - 在 Java 中，`java.lang.Thread` 类封装了底层线程管理逻辑。
    - JVM 使用 JNI（Java Native Interface）调用底层线程管理函数，如 `pthread_create`（Linux）或 `CreateThread`（Windows）。
- **线程调度：** Java 使用操作系统的线程调度策略（通常是抢占式调度），由操作系统的内核线程调度器决定哪个线程运行。

#### **1.2 线程的优先级与调度**

- Java 线程优先级只是对操作系统线程优先级的一个映射，具体效果依赖于操作系统。
- 不同操作系统可能有不同的调度机制：
    - **Linux：** 使用 CFS（完全公平调度器）。
    - **Windows：** 使用时间片轮转调度。

#### **1.3 Java 中的线程模型**

Java 线程模型可以总结为：

- **用户线程：** Java 程序中的 `Thread` 对象。
- **内核线程：** 由操作系统管理，Java 线程与内核线程通常是一对一（1:1）映射关系。



### **2. 线程和系统进程/硬件的对应关系**

#### **2.1 线程和进程的关系**

- **线程是进程的子任务：** 一个进程可以包含多个线程，线程共享进程的地址空间，但每个线程有独立的栈和寄存器状态。
- 在操作系统层面：
    - 每个线程通常有一个 `Thread Control Block (TCB)`，记录线程的寄存器状态、栈指针等信息。
    - 多线程的切换是在同一个进程的上下文内完成的，因此比进程切换更高效。

#### **2.2 硬件和线程的关系**

- 硬件提供多核和超线程支持，为多线程并发执行提供物理基础。
- 操作系统的线程调度程序负责将线程分配到 CPU 核心上运行。

### **3. Go 中的协程与 Java 线程的区别**

Go 的并发模型基于 **goroutines** 和 **GMP 模型**，与 Java 的线程模型有显著不同。

#### **3.1 Go 的协程（goroutine）**

- 轻量级线程：
    - Goroutine 是用户态的协程，由 Go runtime 管理。
    - 每个 Goroutine 只占用少量内存（4~8KB），比操作系统线程更高效。
- 调度模型：GMP
    - G（Goroutine）：表示协程。
    - M（Machine）：表示操作系统线程。
    - P（Processor）：逻辑处理器，用于管理可运行的 G 和 M。
    - Go runtime 的调度器将多个 Goroutines 映射到较少的系统线程上（N:M 模型）。

#### **3.2 Java 线程 vs. Go 协程**

| 特性               | Java 线程              | Go 协程 (goroutine)       |
| ------------------ | ---------------------- | ------------------------- |
| **调度模型**       | 操作系统调度           | Go runtime 调度           |
| **资源占用**       | 重量级（操作系统线程） | 轻量级（用户态协程）      |
| **上下文切换开销** | 较高（内核态切换）     | 较低（用户态切换）        |
| **实现方式**       | 1:1 映射到操作系统线程 | N:M 映射                  |
| **栈内存**         | 通常为 1MB（固定）     | 动态扩展（起始 2KB 左右） |

### **4. 线程与协程的联系和区别**

#### **4.1 联系**

- **并发编程目的相同：** 无论是线程还是协程，目标都是实现并发编程，以提高程序的效率。
- **共享资源：** 无论是线程还是协程，多个执行单元可能共享资源，因此需要同步机制（如锁）来避免竞态条件。

#### **4.2 区别**

| 特性           | 线程                    | 协程                     |
| -------------- | ----------------------- | ------------------------ |
| **调度权**     | 操作系统内核调度        | 用户态程序调度           |
| **切换成本**   | 高（用户态↔内核态切换） | 低（纯用户态切换）       |
| **实现复杂性** | 操作系统负责            | 语言或运行时负责         |
| **并行性支持** | 硬件多核支持真正的并行  | 通常依赖底层线程实现并行 |

#### **4.3 性能权衡**

- **线程适合 I/O 密集型、CPU 密集型任务：** 线程直接依赖操作系统调度，适合需要真实并行的场景。
- **协程适合高并发场景：** 协程由于轻量化和低切换成本，非常适合高并发任务（如处理网络请求）。





