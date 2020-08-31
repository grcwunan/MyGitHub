# <center>Spring听课记录<center>

#### Dispatcher源码分析

​	主要即是DispatcherServlet类中的doDispatch方法：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Object dispatchException = null;

                try {
                    // 检查是否文件上传请求
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    // 决定当前请求用哪个handler(controller)处理
                    mappedHandler = this.getHandler(processedRequest);
                    if (mappedHandler == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }
					
                    // 拿到适配器，适配器(反射工具)与handler的关系
                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = "GET".equals(method);
                    if (isGet || "HEAD".equals(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }
					// 适配器执行目标方法
                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    // 判断是否异步方法
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }
					// 如果没有，则设置默认视图名
                    this.applyDefaultViewName(processedRequest, mv);
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new NestedServletException("Handler dispatch failed", var21);
                }
				// 
                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
            }

        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            } else if (multipartRequestParsed) {
                this.cleanupMultipart(processedRequest);
            }

        }
    }
```

1. 所有请求过来DispatcherServlet收到i请求；

2. 调用doDispatch()方法进行处理：

   1. getHandler()方法
   2. getHandlerAdapter()方法
   3. 使用适配器执行目标方法
   4. 返回一个ModelAndView对象
   5. 根据返回值转发到具体的页面

3. getHandler()，返回HandlerExecutionChain执行链，从this.handlerMappings中遍历请求，找到HandlerMapping；根据当前请求在HandlerMapping中找到这个请求的映射信息，获取到目标处理器

4. getHandlerAdapter()方法，根据当前处理器类，找到HandlerAdapter

5. SpringMVC九大组件：

   ```java
   	/** 文件上传 */
   	@Nullable
   	private MultipartResolver multipartResolver;
   
   	/** 国际化 */
   	@Nullable
   	private LocaleResolver localeResolver;
   
   	/** 主题解析器 */
   	@Nullable
   	private ThemeResolver themeResolver;
   
   	/** List of HandlerMappings used by this servlet. */
   	@Nullable
   	private List<HandlerMapping> handlerMappings;
   
   	/** List of HandlerAdapters used by this servlet. */
   	@Nullable
   	private List<HandlerAdapter> handlerAdapters;
   
   	/** 异常解析 */
   	@Nullable
   	private List<HandlerExceptionResolver> handlerExceptionResolvers;
   
   	/** RequestToViewNameTranslator used by this servlet. */
   	@Nullable
   	private RequestToViewNameTranslator viewNameTranslator;
   
   	/** Flash+Manager: SpringMVC中运行重定向携带数据的功能 */
   	@Nullable
   	private FlashMapManager flashMapManager;
   
   	/** 视图解析器 */
   	@Nullable
   	private List<ViewResolver> viewResolvers;
   ```

   ​	九大组件都是接口，接口即规范；

   ​	在onRefresh()方法中调用initStrategies()方法初始化，onRefresh是Spring框架留出来的方法给子类用的；

   ​	组件初始化：去容器中找到组件，如果没有就使用默认配置，配置文件在mvc的jar包中，DispatcherServlet.properties文件

   ​	ModelAttribute标注的方法比目标方法先运行；

   ​	***TO DO: 各种参数的确定与传输***

![Spring参数确定流程](C:\Users\Administrator\Desktop\临时文件\SpringMVC参数确定流程.JPG)



@ModelAttribute最主要的作用是将数据添加到模型对象中，用于视图页面展示时使用。：

- @ModelAttribute注释方法：被@ModelAttribute注释的方法会在此controller每个方法执行前被执行，因此对于一个controller映射多个URL的用法来说，要谨慎使用。

- @ModelAttribute注释返回具体类的方法：model属性的名称没有指定，它由返回类型隐含表示，如这个方法返回Account类型，那么这个model属性的名称是account。

- @ModelAttribute(value="")注释返回具体类的方法：这个例子中使用@ModelAttribute注释的value属性，来指定model属性的名称。model属性对象就是方法的返回值。它无须要特定的参数。

- @ModelAttribute和@RequestMapping同时注释一个方法：这时这个方法的返回值并不是表示一个视图名称，而是model属性的值，视图名称由RequestToViewNameTranslator根据请求转换为逻辑视图。

- @ModelAttribute注释一个方法的参数：在这个例子里，@ModelAttribute("user") User user注释方法参数，参数user的值来源于addAccount()方法中的model属性。此时如果方法体没有标注@SessionAttributes("user")，那么scope为request，如果标注了，那么scope为session

  ```java
   1 @Controller 
   2 public class HelloWorldController { 
   3     @ModelAttribute("user") 
   4     public User addAccount() { 
   5         return new User("jz","123"); 
   6      } 
   7 
   8     @RequestMapping(value = "/helloWorld") 
   9     public String helloWorld(@ModelAttribute("user") User user) { 
  10            user.setUserName("jizhou"); 
  11            return "helloWorld"; 
  12         } 
  13   }
  ```

- 从Form表单或URL参数中获取（实际上，不做此注释也能拿到user对象）;

- @ModelAttribute注释一个方法的返回值：放在方法的返回值之前，添加方法返回值到模型对象中，用于视图页面展示时使用。

#### SpringMVC的视图解析

​	重定向与转发

​	根据返回值拼页面串

- 任何方法的返回都会包装成ModelAndView对象；
- processDispatchResult()方法来处理：调用render，得到View对象；UrlBasedViewResolver，得到视图对象：
- 调用视图对象的render方法，调用了InternalResourceView.renderMergedOutputModel()
- 视图解析器只是得到视图对象；视图对象才真正的转发或重定向到页面；

#### 国际化

JSTLView，bean的ID必须叫messageSource；

一定要过SpringMVC的视图解析流程，



#### 自定义视图解析器和视图

1. 编写自定义的视图解析器（继承ViewResolver）和视图（继承View）；
2. bean标签注册到IOC容器中，不能让InternalResourceViewResolver拦截到，需要实现Orderd接口，改变视图解析器的优先级；
3. Orderd中数字越小，优先级越高，InternalResourceViewResolver优先级最低；



IDEA快捷键：

- Ctrl+F12: 列出当前类的所有方法
- Ctrl+shift+n: 查找某个类

#### Restful CRUD

Restful风格：

- GET : 查询
- PUT ：修改
- DELETE ：删除
- POST：新增

#### MyBatis配置

两个比较重要的配置文件：

- 全局配置文件（mybatis-config.xml）：知道mybatis正确运行的一些全局设置；

  - [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties)

  - [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings)

  - [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)

  - [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)

  - [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)

  - [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)

    插件是MyBatis提供的一个非常强大的机制，可以通过插件来修改MyBatis的一些核心行为。插件通过动态代理机制，可以介入四大对象的任何一个方法的执行。

    - Executor
    - ParameterHandler
    - ResultSetHandler
    - StatementHandler

  - environments（环境配置）

    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）

  - [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)

    可根据数据库驱动设置执行不同的语句

  - [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers)

- SQL映射文件（EmployeeDao.xml）：相当于是对Dao接口的一个实现描述；

  - `cache` – 该命名空间的缓存配置。
  - `cache-ref` – 引用其它命名空间的缓存配置。
  - `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
  - ~~`parameterMap` – 老式风格的参数映射。此元素已被废弃，并可能在将来被移除！请使用行内参数映射。文档中不会介绍此元素。~~

  - `sql` – 可被其它语句引用的可重用语句块。
  - `insert` – 映射插入语句。
  - `update` – 映射更新语句。
  - `delete` – 映射删除语句。
  - `select` – 映射查询语句。

两个类：SqlSessionFactory和SqlSession

MyBatis缓存机制：Map；

一级缓存：线程级别的缓存，本地缓存，SqlSession级别的缓存；默认存在；查过的数据，MyBtis会缓存在map中；

失效的几种情况：

- SqlSession级别，不同的SqlSession实例，不会使用一级缓存；
- 不同的查询参数
- 执行任何增删改，缓存均清空；
- 手动清空缓存，openSession.clearCache()

二级缓存：全局范围的缓存，其他也可以用；默认不开启，缓存实现要求POJO实现Serializable接口，二级缓存在SqlSession关闭或提交之后才生效；

```xml
<!-- 开启二级缓存 -->
<setting name="cacheEnabled" value="true"/>
```

然后在具体的Dao的xml配置中配置:

```xml
<cache></cache>
```

- namespace级别的缓存，每一个Dao拥有自己的二级缓存；
- 不会出现一级缓存和二级缓存中同时有一条数据？
- 先看二级缓存，再看一级缓存；

#### MBG-逆向工程

根据数据表table，自动生成；



#### 分页插件PageHelper



#### 监听器

- 监听对象
- 触发行为
- 监听事件

八个监听器（JavaWeb的三大组件：servlet、listener、filter）

按照监听对象分为三类：

- ServletContext
- HttpSession
- Servlet request

![](C:\Users\Administrator\Desktop\临时文件\image\Listener分类.JPG)

第一次访问jsp，jsp内置对象session，创建了session



#### 国际化

locale：

ResourceBundle：





# JavaWeb复习

1. html、css、js（结构-表现-行为）
2. js的事件：window.onload 、$(function(){})...
3. jQuery：选择器、文档操作、属性操作
4. 