Q:Session机制
===
![session](https://github.com/IceDarron/Note/blob/master/Image/session_model.png)

假如浏览器A先访问Servlet1，这时候它创建了一个Session，ID号为110，然后Servlet1将这个ID号以Cookie的方式返回给浏览器A，
接着，如果浏览器A继续访问Servlet2，那么这个请求会带上Cookie值: JSESSIONID=110，
然后服务器根据浏览器A传递过来的ID号找到内存中的这个Session。 
这时候假如浏览器B来访问Servlet1了，它的请求并没有带上 JSESSIONID这个Cookie值，
由于它也要使用Session，所以服务器会新创建一个Session，ID号为119，并将这个ID号以Cookie的方式返回给浏览器B。之后的过程就同A了。


Q:Cookie机制
===
当用户使用浏览器访问一个支持Cookie的网站的时候，用户会提供包括用户名在内的个人信息并且提交至服务器；
接着，服务器在向客户端回传相应的超文本的同时也会发回这些个人信息，当然这些信息并不是存放在HTTP响应体（Response Body）中的，
而是存放于HTTP响应头（Response Header）；当客户端浏览器接收到来自服务器的响应之后，浏览器会将这些信息存放在一个统一的位置，
对于Windows操作系统而言，我们可以从： [系统盘]:\Documents and Settings[用户名]\Cookies目录中找到存储的Cookie；
自此，客户端再向服务器发送请求的时候，都会把相应的Cookie再次发回至服务器。
而这次，Cookie信息则存放在HTTP请求头（Request Header）了。
有了Cookie这样的技术实现，服务器在接收到来自客户端浏览器的请求之后，就能够通过分析存放于请求头的Cookie得到客户端特有的信息，
从而动态生成与该客户端相对应的内容。


Q:分布式Session的几种实现方式
===
+ 基于数据库的Session共享
+ 基于NFS共享文件系统
+ 基于memcached 的session，如何保证 memcached 本身的高可用性？
+ 基于resin/tomcat web容器本身的session复制机制
+ 基于TT/Redis 或 jbosscache 进行 session 共享。
+ 基于cookie 进行session共享


Q:web应用中的Filter、Servlet、Listener
===
web.xml 的加载顺序是：context-param -> listener -> filter -> servlet ，
而同个类型之间的实际程序调用的时候的顺序是根据对应的 mapping 的顺序进行调用的。

### Web.xml常用元素
```xml
<web-app>    
<display-name></display-name>定义了WEB应用的名字    
<description></description> 声明WEB应用的描述信息    

<context-param></context-param> context-param元素声明应用范围内的初始化参数。    
<filter></filter> 过滤器元素将一个名字与一个实现javax.servlet.Filter接口的类相关联。    
<filter-mapping></filter-mapping> 一旦命名了一个过滤器，就要利用filter-mapping元素把它与一个或多个servlet或JSP页面相关联。    
<listener></listener>servlet API的版本2.3增加了对事件监听程序的支持，事件监听程序在建立、修改和删除会话或servlet环境时得到通知。    
                     Listener元素指出事件监听程序类。    
<servlet></servlet> 在向servlet或JSP页面制定初始化参数或定制URL时，必须首先命名servlet或JSP页面。Servlet元素就是用来完成此项任务的。    
<servlet-mapping></servlet-mapping> 服务器一般为servlet提供一个缺省的URL：http://host/webAppPrefix/servlet/ServletName。    
              但是，常常会更改这个URL，以便servlet可以访问初始化参数或更容易地处理相对URL。在更改缺省URL时，使用servlet-mapping元素。    

<session-config></session-config> 如果某个会话在一定时间内未被访问，服务器可以抛弃它以节省内存。    
          可通过使用HttpSession的setMaxInactiveInterval方法明确设置单个会话对象的超时值，或者可利用session-config元素制定缺省超时值。    

<mime-mapping></mime-mapping>如果Web应用具有想到特殊的文件，希望能保证给他们分配特定的MIME类型，则mime-mapping元素提供这种保证。    
<welcome-file-list></welcome-file-list> 指示服务器在收到引用一个目录名而不是文件名的URL时，使用哪个文件。    
<error-page></error-page> 在返回特定HTTP状态代码时，或者特定类型的异常被抛出时，能够制定将要显示的页面。    
<taglib></taglib> 对标记库描述符文件（Tag Libraryu Descriptor file）指定别名。此功能使你能够更改TLD文件的位置，    
                  而不用编辑使用这些文件的JSP页面。    
<resource-env-ref></resource-env-ref>声明与资源相关的一个管理对象。    
<resource-ref></resource-ref> 声明一个资源工厂使用的外部资源。    
<security-constraint></security-constraint> 制定应该保护的URL。它与login-config元素联合使用    
<login-config></login-config> 指定服务器应该怎样给试图访问受保护页面的用户授权。它与sercurity-constraint元素联合使用。    
<security-role></security-role>给出安全角色的一个列表，这些角色将出现在servlet元素内的security-role-ref元素    
                   的role-name子元素中。分别地声明角色可使高级IDE处理安全信息更为容易。    
<env-entry></env-entry>声明Web应用的环境项。    
<ejb-ref></ejb-ref>声明一个EJB的主目录的引用。    
<ejb-local-ref></ ejb-local-ref>声明一个EJB的本地主目录的应用。    
</web-app>    
```

### 相应元素配置    
+ Web应用图标：指出IDE和GUI工具用来表示Web应用的大图标和小图标   
```xml
<icon>    
	<small-icon>/images/app_small.gif</small-icon>    
	<large-icon>/images/app_large.gif</large-icon>    
</icon>
```

+ Web 应用名称：提供GUI工具可能会用来标记这个特定的Web应用的一个名称    
```xml
<display-name>Tomcat Example</display-name>
```

+ Web 应用描述： 给出于此相关的说明性文本
```xml
<disciption>Tomcat Example servlets and JSP pages.</disciption>    
```

+ 上下文参数：声明应用范围内的初始化参数。    
```xml
<context-param>    
    <param-name>ContextParameter</para-name>    
    <param-value>test</param-value>    
    <description>It is a test parameter.</description>    
</context-param>    
```
在servlet里面可以通过getServletContext().getInitParameter("context/param")得到    

+ 过滤器配置：将一个名字与一个实现javaxs.servlet.Filter接口的类相关联。    
```xml
<filter>    
        <filter-name>setCharacterEncoding</filter-name>    
        <filter-class>com.myTest.setCharacterEncodingFilter</filter-class>    
        <init-param>    
            <param-name>encoding</param-name>    
            <param-value>GB2312</param-value>    
        </init-param>    
</filter>    
<filter-mapping>    
        <filter-name>setCharacterEncoding</filter-name>    
        <url-pattern>/*</url-pattern>    
</filter-mapping>
```
    
+ 监听器配置    
```xml
<listener>    
      <listerner-class>listener.SessionListener</listener-class>    
</listener>    
```

+ Servlet配置  
基本配置  
```xml
<servlet>    
  <servlet-name>snoop</servlet-name>    
  <servlet-class>SnoopServlet</servlet-class>    
</servlet>    
<servlet-mapping>    
  <servlet-name>snoop</servlet-name>    
  <url-pattern>/snoop</url-pattern>    
</servlet-mapping>    
```

高级配置    
```xml
<servlet>    
  <servlet-name>snoop</servlet-name>    
  <servlet-class>SnoopServlet</servlet-class>    
  <init-param>    
     <param-name>foo</param-name>    
     <param-value>bar</param-value>    
  </init-param>    
  <run-as>    
     <description>Security role for anonymous access</description>    
     <role-name>tomcat</role-name>    
  </run-as>    
</servlet>    
<servlet-mapping>    
  <servlet-name>snoop</servlet-name>    
  <url-pattern>/snoop</url-pattern>    
</servlet-mapping> 
```
   
元素说明 
```xml   
<servlet></servlet> 用来声明一个servlet的数据，主要有以下子元素：    
<servlet-name></servlet-name> 指定servlet的名称    
<servlet-class></servlet-class> 指定servlet的类名称    
<jsp-file></jsp-file> 指定web站台中的某个JSP网页的完整路径    
<init-param></init-param> 用来定义参数，可有多个init-param。在servlet类中通过getInitParamenter(String name)方法访问初始化参数    
<load-on-startup></load-on-startup>指定当Web应用启动时，装载Servlet的次序。    
                         当值为正数或零时：Servlet容器先加载数值小的servlet，再依次加载其他数值大的servlet.    
                         当值为负或未定义：Servlet容器将在Web客户首次访问这个servlet时加载它    
<servlet-mapping></servlet-mapping> 用来定义servlet所对应的URL，包含两个子元素    
<servlet-name></servlet-name> 指定servlet的名称    
<url-pattern></url-pattern> 指定servlet所对应的URL   
```
 
+ 会话超时配置（单位为分钟） 
```xml   
<session-config>    
  <session-timeout>120</session-timeout>    
</session-config>    
```

+ MIME类型配置    
```xml
<mime-mapping>    
  <extension>htm</extension>    
  <mime-type>text/html</mime-type>    
</mime-mapping>    
```

+ 指定欢迎文件页配置 
```xml   
<welcome-file-list>    
  <welcome-file>index.jsp</welcome-file>    
  <welcome-file>index.html</welcome-file>    
  <welcome-file>index.htm</welcome-file>    
</welcome-file-list> 
```   

+ 配置错误页面    
通过错误码来配置error-page  
```xml  
<error-page>    
  <error-code>404</error-code>    
  <location>/NotFound.jsp</location>    
</error-page>   
```
上面配置了当系统发生404错误时，跳转到错误处理页面NotFound.jsp。  
  
通过异常的类型配置error-page    
```xml
<error-page>    
   <exception-type>java.lang.NullException</exception-type>    
   <location>/error.jsp</location>    
</error-page>    
```
上面配置了当系统发生java.lang.NullException（即空指针异常）时，跳转到错误处理页面error.jsp    

+ TLD配置    
```xml
<taglib>    
   <taglib-uri>http://jakarta.apache.org/tomcat/debug-taglib</taglib-uri>    
   <taglib-location>/WEB-INF/jsp/debug-taglib.tld</taglib-location>    
</taglib>    

如果MyEclipse一直在报错,应该把<taglib> 放到 <jsp-config>中   
	 
<jsp-config>    
  <taglib>    
      <taglib-uri>http://jakarta.apache.org/tomcat/debug-taglib</taglib-uri>    
      <taglib-location>/WEB-INF/pager-taglib.tld</taglib-location>    
  </taglib>    
</jsp-config>    
```

+ 资源管理对象配置 
```xml   
<resource-env-ref>    
   <resource-env-ref-name>jms/StockQueue</resource-env-ref-name>    
</resource-env-ref>  
```
  
+ 资源工厂配置  
```xml  
<resource-ref>    
   <res-ref-name>mail/Session</res-ref-name>    
   <res-type>javax.mail.Session</res-type>    
   <res-auth>Container</res-auth>    
</resource-ref>    
配置数据库连接池就可在此配置：    
<resource-ref>    
   <description>JNDI JDBC DataSource of shop</description>    
   <res-ref-name>jdbc/sample_db</res-ref-name>    
   <res-type>javax.sql.DataSource</res-type>    
   <res-auth>Container</res-auth>    
</resource-ref>   
```
 
+ 安全限制配置    
```xml
<security-constraint>    
  <display-name>Example Security Constraint</display-name>    
  <web-resource-collection>    
     <web-resource-name>Protected Area</web-resource-name>    
     <url-pattern>/jsp/security/protected/*</url-pattern>    
     <http-method>DELETE</http-method>    
     <http-method>GET</http-method>    
     <http-method>POST</http-method>    
     <http-method>PUT</http-method>    
  </web-resource-collection>    
  <auth-constraint>    
    <role-name>tomcat</role-name>    
    <role-name>role1</role-name>    
  </auth-constraint>    
</security-constraint>  
```
  
+ 登陆验证配置    
```xml
<login-config>    
 <auth-method>FORM</auth-method>    
 <realm-name>Example-Based Authentiation Area</realm-name>    
 <form-login-config>    
    <form-login-page>/jsp/security/protected/login.jsp</form-login-page>    
    <form-error-page>/jsp/security/protected/error.jsp</form-error-page>    
 </form-login-config>    
</login-config>  
```
  
+ 安全角色：security-role元素给出安全角色的一个列表，这些角色将出现在servlet元素内的security-role-ref元素的role-name子元素中。    
    分别地声明角色可使高级IDE处理安全信息更为容易。    
```xml
<security-role>    
     <role-name>tomcat</role-name>    
</security-role>   
```
 
+ Web环境参数：env-entry元素声明Web应用的环境项 
```xml   
<env-entry>    
     <env-entry-name>minExemptions</env-entry-name>    
     <env-entry-value>1</env-entry-value>    
     <env-entry-type>java.lang.Integer</env-entry-type>    
</env-entry>    
```

+ EJB 声明   
```xml 
<ejb-ref>    
     <description>Example EJB reference</decription>    
     <ejb-ref-name>ejb/Account</ejb-ref-name>    
     <ejb-ref-type>Entity</ejb-ref-type>    
     <home>com.mycompany.mypackage.AccountHome</home>    
     <remote>com.mycompany.mypackage.Account</remote>    
</ejb-ref>   
```
 
+ 本地EJB声明    
```xml
<ejb-local-ref>    
     <description>Example Loacal EJB reference</decription>    
     <ejb-ref-name>ejb/ProcessOrder</ejb-ref-name>    
     <ejb-ref-type>Session</ejb-ref-type>    
     <local-home>com.mycompany.mypackage.ProcessOrderHome</local-home>    
     <local>com.mycompany.mypackage.ProcessOrder</local>    
</ejb-local-ref>    
```

+ 配置DWR 
```xml   
<servlet>    
      <servlet-name>dwr-invoker</servlet-name>    
      <servlet-class>uk.ltd.getahead.dwr.DWRServlet</servlet-class>    
</servlet>    
<servlet-mapping>    
      <servlet-name>dwr-invoker</servlet-name>    
      <url-pattern>/dwr/*</url-pattern>    
</servlet-mapping>   
```
 
+ 配置Struts    
```xml
<display-name>Struts Blank Application</display-name>    
<servlet>    
    <servlet-name>action</servlet-name>    
    <servlet-class>    
        org.apache.struts.action.ActionServlet    
    </servlet-class>    
    <init-param>    
        <param-name>detail</param-name>    
        <param-value>2</param-value>    
    </init-param>    
    <init-param>    
        <param-name>debug</param-name>    
        <param-value>2</param-value>    
    </init-param>    
    <init-param>    
        <param-name>config</param-name>    
        <param-value>/WEB-INF/struts-config.xml</param-value>    
    </init-param>    
    <init-param>    
        <param-name>application</param-name>    
        <param-value>ApplicationResources</param-value>    
    </init-param>    
    <load-on-startup>2</load-on-startup>    
</servlet>    
<servlet-mapping>    
    <servlet-name>action</servlet-name>    
    <url-pattern>*.do</url-pattern>    
</servlet-mapping>    
<welcome-file-list>    
    <welcome-file>index.jsp</welcome-file>    
</welcome-file-list>    

<!-- Struts Tag Library Descriptors -->    
<taglib>    
    <taglib-uri>struts-bean</taglib-uri>    
    <taglib-location>/WEB-INF/tld/struts-bean.tld</taglib-location>    
</taglib>    
<taglib>    
    <taglib-uri>struts-html</taglib-uri>    
    <taglib-location>/WEB-INF/tld/struts-html.tld</taglib-location>    
</taglib>    
<taglib>    
<taglib-uri>struts-nested</taglib-uri>    
<taglib-location>/WEB-INF/tld/struts-nested.tld</taglib-location>    
</taglib>    
<taglib>    
    <taglib-uri>struts-logic</taglib-uri>    
    <taglib-location>/WEB-INF/tld/struts-logic.tld</taglib-location>    
</taglib>    
<taglib>    
    <taglib-uri>struts-tiles</taglib-uri>    
    <taglib-location>/WEB-INF/tld/struts-tiles.tld</taglib-location>    
</taglib>   
```
 
+ 配置Spring（基本上都是在Struts中配置的）    
```xml
<!-- 指定spring配置文件位置 -->    
<context-param>    
  <param-name>contextConfigLocation</param-name>    
  <param-value>    
   <!--加载多个spring配置文件 -->    
    /WEB-INF/applicationContext.xml, /WEB-INF/action-servlet.xml    
  </param-value>    
</context-param>    

<!-- 定义SPRING监听器，加载spring -->    

<listener>    
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>    
</listener>    

<listener>    
     <listener-class>    
       org.springframework.web.context.request.RequestContextListener    
     </listener-class>    
</listener>
```

+ 配置springMVC
```xml
<!-- 由tomcat加载spring容器，加入Spring相关配置 -->
<context-param>
　　<param-name>contextConfigLocation</param-name>
　　<param-value>classpath:spring_config/applicationContext.xml</param-value>
</context-param>
<listener>
　　<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>


<filter>
　　<description>字符集过滤器</description>
　　<filter-name>encodingFilter</filter-name>
　　<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
　　<init-param>
　　　　<description>字符集编码</description>
　　　　<param-name>encoding</param-name>
　　　　<param-value>UTF-8</param-value>
　　</init-param>
</filter>
<filter-mapping>
　　<filter-name>encodingFilter</filter-name>
　　<url-pattern>/*</url-pattern>
</filter-mapping>

<!-- Spring MVC 相关配置 -->
<servlet>
　　<servlet-name>Dispatcher</servlet-name>
　　<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
　　<init-param>
　　　　<param-name>contextConfigLocation</param-name>
　　　　<param-value>classpath:spring_config/applicationContext-mvc.xml</param-value>
　　</init-param>

　　<load-on-startup>1</load-on-startup>
</servlet>
<!--为DispatcherServlet建立映射 --> 
<servlet-mapping>
　　<servlet-name>Dispatcher</servlet-name>
　　<url-pattern>*.do</url-pattern>
</servlet-mapping>
```
