
## 杂记

#### 为什么要创建war包？

1.  ==主要原因==：创建WAR(Web Application Archive)包是为了将Java Web应用程序 **部署**到传统的Servlet容器（Tomcat、jetty、wildfly等）中。

2.  war包是一个压缩文件，包含了应用程序运行所需要的所有内容：

    *   **便于版本管理**：可以轻松地将不同版本的 WAR 包部署到服务器上，方便进行版本回滚和升级。

    *   **便于移植**：一个 WAR 包可以部署到任何符合 Java EE 标准的 Servlet 容器中，保证应用程序的可移植性。

#### 日志的重要性(现在没有体会)

#### Maven的pom.xml文件中标签的作用

*   它定义了该**依赖**在项目构建的各个阶段（如编译、测试、运行、打包等）中的可见性和作用域。不同的依赖范围决定了依赖库是否包含在编译时、测试时、运行时以及打包时。

##### 作用范围

**编译时可用**：编译主源码时需要用到。

**测试时可用**：在测试时同样需要用到。

**运行时可用**：程序运行时也需要用到。

**打包时可用**：会被打包到最终的可执行文件（如 WAR 或 JAR）中。

*   常用依赖范围：

    1.  compile(默认)：表示依赖在编译、测试、运行和打包的所有阶段都可用。

    2.  provided：依赖在编译和测试时可用，运行时、打包时不可用。     但在运行时需要由目标环境（如应用服务器）提供，打包时不会被包含在最终的可执行文件中。

    3.  runtime：在编译时不需要，在其他过程需要。 eg:JDBC驱动程序，反射加载

    4.  test：只在测试时可用。

    5.  system：

    6.  import(目前用不到)：只能在 `<dependencyManagement>` 中使用，用于导入一个依赖的 **BOM**（Bill of Materials），它会将所有该 BOM 中定义的依赖作为当前项目的依赖管理。   eg:用于引入 Spring Boot 的 BOM 或其他父级项目的依赖版本管理

#### servlet用于处理请求的一些问题

##### servlet与路径映射

*   浏览器发出的请求(  用户访问某个URL时  )不能直接访问类。所以要访问servlet，必须要在xml中给请求设置访问路径。当前请求才能被该servlet处理。

*   如果在web.xml文件中注册servelt，需要通过中的来设置映射路径。通过配置不同的urlpatterns，可将不同的url请求分发给不同的servlet来处理。

*   ==匹配优先级==：当多个 `url-pattern` 同时匹配一个请求时，会按照以下优先级来选择处理程序

    1.  **精确匹配**：比如 `/index.jsp` 精确匹配 `index.jsp`。

    2.  .**最长路径匹配（路径模式）**：比如 `/user/*` 比 `/user` 优先匹配。

    3.  **扩展名匹配**：比如 `*.jsp`。

    4.  **根路径匹配**：即 `/`，优先级最低。

*   `/` 和 `/\*` 的实际使用区别：

1.  `/*` 匹配所有请求（包括静态资源）。

2.  `/` (默认Servlet)匹配未被其他更具体的 `url-pattern` 映射的请求（通常不处理静态资源）。 优先级较低。

##### 在web.xml中配置

    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <!-- 给DispatcherServelt配置初始化参数
    这个contextConfigLocation的值来指定了SpringMVC配置文件的具体位置
    -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springMVC.xml</param-value>
        </init-param>
        
        
     <!-- 可以指定Servlet的启动加载顺序。当≥0时，会在启动项目时加载，数值越小越先加载。 ＜0时，会在第一次请求访问时加载
    -->   
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

*   整体逻辑：

1.  \*\*配置 \*\*`DispatcherServlet`: 使用 Spring MVC 的 `DispatcherServlet` 作为前端控制器，所有 HTTP 请求都会先由这个 `Servlet` 接管。

2.  **指定配置文件位置**: `DispatcherServlet` 在初始化时会从 `classpath:springMVC.xml` 文件中加载 Spring MVC 的相关配置。

3.  **请求分发**: `DispatcherServlet` 会根据 URL 模式 `/` 拦截所有请求，并根据 `springMVC.xml` 中的配置，将请求转发给合适的控制器（Controller）进行处理。

4.  **应用初始化顺序**: 该 `Servlet` 在应用启动时就会被初始化，因为 `load-on-startup` 属性设置为 `1`。

#### IOC容器组件(Bean)

IOC 容器组件，也称为 Bean，是 Spring 框架中的核心单元。IOC 容器**管理的对象**被称为 Bean，每个 Bean 都有唯一的标识符，并且在容器中有自己独立的生命周期和作用范围（Scope）。

#### 在一个SpringMVC项目中，为什么是在springMVC.XML中进行扫描配置，而不是在web.XML中配置

##### 职责分离原则

`web.xml` 是标准的 Java EE 部署描述符文件，主要用于配置 Servlet、过滤器（Filter）、监听器（Listener）等与 Web 容器直接相关的内容。它的职责主要是管理应用程序启动和请求分发等 Web 层面的配置。

而 `springMVC.xml` 文件是 Spring MVC 特有的配置文件，用于配置 Spring MVC 的上下文（WebApplicationContext），它的主要职责是管理 MVC 层（如控制器、视图解析器、消息转换器等）的相关配置，以及相关组件的扫描配置。使用不同的配置文件可以很好地将 Web 层配置与应用层逻辑分开：

*   **web.xml** 中的配置：配置 Web 层面的内容，如 `DispatcherServlet` 的初始化、URL 映射等。

*   **springMVC.xml** 中的配置：配置 Spring MVC 框架特有的内容，如控制器（Controller）、服务（Service）、数据访问层（DAO）、视图解析器等。

通过这种职责分离，可以将 Web 层和应用逻辑层的配置文件分开管理，增强了应用的可维护性和模块化。

##### 加载顺序

*   `web.xml` 加载: `web.xml` 是 Web 容器启动时首先加载的配置文件，负责启动 `DispatcherServlet` 及其他 Servlet、Filter 等组件。

*   `springMVC.xml` 加载: `springMVC.xml` 是由 `DispatcherServlet` 在初始化时加载的配置文件，加载顺序在 `web.xml` 之后。

这种顺序有助于确保在 Web 应用启动时，`DispatcherServlet` 会优先加载，并且可以根据 `web.xml` 中配置的上下文参数来加载对应的 Spring MVC 配置文件。
