## Spring Boot知识点
### Spring Boot和Spring框架的关系
- #### Spring框架：通过IOC和AOP等技术实现的一个应用框架和依赖反转容器，上层提供对应功能模块，按功能区分大致有：Spring AOP、Spring MVC、Spring Security、Spring ORM、Spring JDBC、Spring Test等。Spring项目启动时需要引入大量依赖包和配置若干模板配置文件。
- #### Spring Boot：依赖Spring框架，对其进行扩展和封装的应用框架，继承Spring框架绝大部分原有特性，提供了更多的特性，大致有：
    - 提供多种Starter组件，便于处理组件依赖和配置
    - 内置servlet运行容器，减少部署复杂度
    - 运行指标和项目健康度检查等多种功能
    - 自动配置与检查
### Spring Boot启动
##### 和原始Spring项目只能通过servlet容器启动不同，Spring Boot项目也可以通过项目main方法启动：当项目通过内置servlet容器启动时，项目通过main方法作为启动入口；当项目通过外置容器启动时，通过@SpringBootApplication注解所在类继承SpringBootServletInitializer类，从重写的configure方法中加载启动资源。

### Spring Boot集成和配置
- #### 按需求引入starter-parent，特别的，如果有自定义父pom模块，可以把starter-parent放到dependency-management中
- #### 引入用到的starter组件和相关数据库等组件，starter-web中内置Tomcat，如果需要切换则排除掉，修改build-plugin
- #### 创建启动类，根据需要配置相应注解，对需要打成war包外部启动的，继承Spring Boot的servlet启动类，重写相应方法
- #### 根据需要配置相关bean或configuration，比如自定义过滤器；有需要在系统启动后执行的CommandLineRunner；配置系统exitCode等
- #### 创建对应资源目录，配置对应配置项，如数据库相关参数、运行检查、启动路径/端口、日志文件等
#### 列举了常见集成配置项，需求和引入模块不同配置细节也有区别。
### 常用starter
#### Spring Boot根据功能块不同，提供独立的启动模块，启动模块引入了依赖组件，可以按需替换，常用几种模块为:
   - > spring-boot-starter:基本starter，提供spring-boot,spring-context,spring-beans基础模块。
   - > starter-web:提供控制层开发功能，针对springs，引入了pring-webmvc、spring-web组件依赖，另外也引入了内置Tomcat和Jackson等包。特别地，自动构建了servlet-dispatcher和视图解析器等原来需要手动配置的功能。
    - > data-jpa：基于spring-data-jap的对象关联映射框架，引入了spring-orm、hibernate-entity-manager、spring-data-jpa组件。
    - > starter-jdbc：提供数据源装配、提供jdbcTemplate供简化使用、提供事务。
    - > starter-security：对spring-security提供依赖封装
    - > starter-test：提供常见框架支持，如Junit、Mockito等
#### 以上只列举了常用的几种，根据功能其他还有很多种，比如starter-activemq、starter-validation。
### Spring Boot CLI
##### CLI是Spring Boot的命令行操作工具，通过借助groovy脚本，可以使用命令对项目进行初始化、管理组件依赖、打包等操作。
### starter加载原理和自定义
#### 官方提供了若干starter，能满足大部分使用需求，但对于特殊场景我们需要按需加载某类或使用某功能时或者官方提供的组件有依赖或配置实际不需要时，可以自定义starter。
#### Spring Boot项目使用@SpringBootApplication注解作为启动类，该注解包含@EnableAutoConfiguration注解，而后者使用@Import导入EnableAutoConfigurationImportSelector类，后者内部扫描了包含META-INF/spring.factories文件的jar包，而spring.factories中以key-value存放项目所有需要加载的AutoConfigure类。因此对应的，我们通过自定义类实现所要加载的类，通过配置@configuration和其他辅助注解，打成jar包，将自定义类配置到srping.factories文件中，并在pom中引入自定义jar包就可以实现了。注意点是自定义jar包命名必须按照Spring Boot规范。
 

