# SpringMVC

自从使用上了Spring Boot，感觉对Spring MVC距离越来越远了，今天打开以前写的老demo，竟然项目启动不起来，排查问题也排查了好久，一怒之下，开始回顾下Spring MVC的一些知识点吧。毕竟Spring Boot是依赖于Spring+Spring MVC的，连Spring MVC都搞不透彻，实在有点儿过意不去！！！

## 1.Spring MVC如何使用纯java的配置方式？

1. 定义Spring MVC的配置类，它是Spring应用容器的配置文件

   ```java
   @Configuration
   @EnableWebMvc
   @ComponentScan(basePackages = {"com.lee.javaconfig.controller"})
   public class WebConfig extends WebMvcConfigurerAdapter {
   
       @Override
       public void configureViewResolvers(ViewResolverRegistry registry) {
           super.configureViewResolvers(registry);
           InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
           viewResolver.setPrefix("/jsp");
           viewResolver.setSuffix(".jsp");
           registry.viewResolver(viewResolver);
       }
   
       /**
        * 添加静态资源映射
        *
        * @param registry
        */
       @Override
       public void addResourceHandlers(ResourceHandlerRegistry registry) {
           super.addResourceHandlers(registry);
           registry.addResourceHandler("/css/**").addResourceLocations("/static/css/");
       }
   
       /**
        * 添加json转换器,注意如果使用configureMessageConverters()方法,则会清空以前的转换器
        *
        * @param converters
        */
       @Override
       public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
           FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
           converters.add(converter);
       }
   
       // 配置静态文件处理,DispatcherServlet将会把针对静态资源的请求转交给servlet容器的default servlet处理
       @Override
       public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
           configurer.enable();
       }
   }
   ```

   

2. 定义Spring容器的配置类，它是Spring MVC应用容器的配置文件，配置basePackages时，选择service、component等所在的包即可

   ```java
   @Configuration
   @ComponentScan(basePackages = {"com.lee.javaconfig.service"})
   public class RootConfig {
   }
   ```

   

3. 继承`AbstractAnnotationConfigDispatcherServletInitializer`来负责配置DispatcherServlet、初始化SpringMVC容器和Spring容器

   ```java
   public class WebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
       @Override
       protected Class<?>[] getRootConfigClasses() {
           return new Class[]{RootConfig.class};
       }
   
       @Override
       protected Class<?>[] getServletConfigClasses() {
           return new Class[]{WebConfig.class};
       }
   
       /**
        * "/"与"/*"的区别:
        * 1./会匹配到/hello这种RESTFul风格的url,不会匹配到*.jsp这样类型的url
        * 2./*会匹配到所有的路径型(/hello)和后缀型的url(hello.jsp)
        * 为什么使用/*会出现404?
        * 例如浏览器请求:http://localhost:8080/hello 前端控制器查找到了控制器,然后视图解析器查询到了/jsp/hello.jsp这个页面,然后跳转过去,此时又被
        * dispatcher拦截下来了,前端控制器继续查找控制器,此时没有找到了,因此报错404(我的没有报404但是返回的是jsp文件的源代码文件,哈哈,如果jsp文件放在WEB-INF下面就会报404啦,放在webapp下面,是直接不用controller直接访问到的)
        * @return
        */
       @Override
       protected String[] getServletMappings() {
           return new String[]{"/"};
       }
   }
   ```

   说明:

   `getRootConfigClasses()`：方法用于获取Spring应用容器的配置文件 

   `getServletConfigClasses `：负责获取Spring MVC应用容器 

   `getServletMappings()`：负责指定需要由DispatcherServlet映射的路径，我们这里给定的是`/`，意思是DispatcherServlet处理所有的RESTFul风格的请求（不包含/xx/xx.jsp这类带后缀的请求，目前这种指定方式越来越请求，如果存在静态资源不由DispatcherServlet来请求的话，再配置一个静态资源处理器即可，见步骤1中的`addResourceHandlers`这个方法）

   

   ## 2.jsp文件放在WEB-INF文件夹下和webroot（webapp）有什么区别？

   jsp放在在webroot目录下（IDEA中构建web应用是webapp目录下），这样可以让用户直接访问；jsp放在WEB-INF目录下，就必须通过请求（Spring MVC中需要通过handler转发才能访问）。因此可见放在WEB-INF下会更加安全一些。其实，通过过滤器（拦截器），放在webroot下面的文件也可以实现不能直接访问。

   

   ## 3.说说redirect和forward的区别

   ```java
   response.sendRedirect(url);
   request.getRequestDispatcher().forward(request,response);
   ```

   我们知道JSP中这两个对象都可以使页面跳转，但是二者之间有很大的区别，主要有以下几点：

   1. redirect：重定向到指定的url；forward：请求转发到指定的url
   2. redirect：是客户端跳转；forward：是服务端跳转
   3. redirect：跳转到指定url以后，上个页面（跳转之前的原来页面）中的请求全部结束，原request对象将会消亡，数据也会消亡，紧接着，当前新页面会新建request对象，即产生新的request对象；`request.getRequestDispatcher(url).forward(request,response) `是采用请求转发的方式，在跳转页面的时候带着原来页面的request和response跳转的，request对象始终存在，不会重新创建
   4. 使用response。sendRedirect（）地址栏中的地址会改变；使用request.getRequestDispatcher().forward(request,response)地址栏中的地址不会发生改变(服务端偷梁换柱了，浏览器肯定是不知道的)
   5. 使用redirect想向下个页面传递参数，只能在url后面拼接上参数，例如：`login?username=lee&password=123`；使用forward想向下个页面传递参数，可以设置`request.setAttribute("username","lee")`。
   6. 使用sendRedirect方法可以让你重定向到任何的URL

   

   

   参考：

   Spring实战5-基于Spring构建Web应用 <https://segmentfault.com/a/1190000004343063?_ea=575820> 

   深入理解SpringMVC <https://mp.weixin.qq.com/s/r8CSLRpmYzC263bIElbWCw> 

   

   