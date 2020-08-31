# JavaWeb听课笔记

## Servlet

常用类：

- HttpServletRequest

请求转发：req.forward(req, resp)：浏览器地址不变，请求域req为同一个，可以转发到WEB-INF下，不能转发到外部工程；

base标签：

```xml
<base href="" />
```

- HttpServletResponse类：

  getOutputStream();

  getWriter();

  两个只能使用一个

Servlet实现重定向：

- 设置状态码302；resp.setStatus(302)
- 设置Location，新地址；resp.setAttribute("Location","http://...")

第二种方式：resp.sendRedirect()

特点：

1. 浏览器地址发生变化；
2. 不共享Request域了；
3. 浏览器发送了多次；
4. 不能访问WEB-INF下的资源；
5. 可以访问工程外的资源；

##  JSP

Servlet回传数据非常麻烦；

jsp页面的本质是一个Servlet程序；第一次访问jsp页面，tomcat会生成java类和class字节码，继承了HttpServlet

JSP头部的page指令：各种属性

## HttpSession

1. 是一个接口；
2. Session就是会话；
3. 每个客户端都有自己的一个Session会话；
4. Session常保存在服务器端，经常用来保存用户的登录信息；
5. request.getSession()，第一次调用时创建，之后调用都是获取；isNew()判断是否新创建的
6. 底层是通过Cookie来实现的；

## Filter

## json

json两种存在形式：

- 对象的形式；JSON.stringify()
- 字符串形式；JSON.parse()

## AJAX

ajax是一种浏览器通过js异步发起请求，局部更新页面的技术。

步骤：

1. 创建XHR（XMLHttpRequest）
2. open方法
3. send方法
4. 