---
layout: post
title: "spring boot集成security"
date: 2020-03-20 20:59:04 +0800
categories: [security, security入门, security方法保护, security集成jpa, security集成jpa的坑]
description: "本文详细介绍Spring Security的集成与使用，涵盖安全框架的选择、Spring Security的特点、配置步骤、方法级保护及数据库用户认证。通过实例演示了如何配置WebSecurityConfigurerAdapter，实现用户认证、授权和界面保护。"
keywords: security, security入门, security方法保护, security集成jpa, security集成jpa的坑
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104974151
> - 发布时间：2020-03-20 20:59:04
> - 阅读量：588
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#security, #security入门, #security方法保护, #security集成jpa, #security集成jpa的坑

## 摘要

文章浏览阅读588次。本文详细介绍Spring Security的集成与使用，涵盖安全框架的选择、Spring Security的特点、配置步骤、方法级保护及数据库用户认证。通过实例演示了如何配置WebSecurityConfigurerAdapter，实现用户认证、授权和界面保护。

---

#### spring boot集成security

  * 1\. security介绍
  * 2 为什么选择 Spring Security
  * 3\. Security如何使用
  *     * 3.1 创建
    * 3.2 配置
    * 3.3 security 配置
    *       * 3.3.1 configureGlobal方法
      * 3.3.2 启动登陆
      * 3.3.3 自定义配置 configure
      * 3.3.4 controller
      * 3.3.5 界面
    * 3.4 启动
  * 4\. security 方法保护
  *     * 4.1 创建实体
    * 4.2 创建服务
    * 4.3 创建controller
    * 4.4 访问验证
  * 5\. 从数据库中读取用户认证信息
  *     * 5.1 创建
    * 5.2 配置
    * 5.3 创建实体
    * 5.4 dao
    * 5.5 service
    * 5.6 config
    * 5.7 启动验证
  * 6\. 总结

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. security介绍

Spring Security 是Spring Resource 社区的一个安全组件.Sping Secuity为JavaEE企业级开发提供了全面的安全防护，安全防护是一个不断变化的目标，Spring Security通过版本不断迭代来实现这一目标。Spine Sceunt采用"安全层”的概念，使每一层都尽可能安全，连续的安全层可以达到全面的防护。Spring Security可以在Contoller层、Service层，DAO层等以加注解的方式来保护应用程序的安全，Spring Security 提供了细粒度的权限控制，可以精细到每一个API接口、每一个业务的方法，或者每一个操作数据库的DAO层的方法.Spring Security提供的是应用程序层的安全解决方案，一个系统的安全还需要考患传输层和系统层的安全，例如采用Htpps协议、服务器部署防火墙等。

## 2 为什么选择 Spring Security

使用 Spring Securiy有很多原因，其中一个重要原因是它对环境的无依赖性、低代码耦合性。将工程重现部署到一个新的服务器上，不需要为 Spring Security做什么工作。Spring Security 提供了数十个安全模块，模块与模块间的耦合性低，模块之间可以自由组合来实现特定需求的安全功能，具有较高的可定制性。总而言之，Spring Security 具有很好的可复用性和可定制性。  
在安全方面，有两个主要的领域，一是“认证”，即你是谁；二是“授权”，即你拥有什么权限，Spring Security 的主要目标就是在这两个领域。“认证”是认证主体的过程，通常是指可以在应用程序中执行操作的用户、设备或其他系统。“授权”是指决定是否允许已认证的主体执行某一项操作。  
安全框架多种多样，那为什么选择 Spring Security 作为微服务开发的安全框架呢?JavaEE 有另一个优秀的安全框架 Apache Shiro，Apache Shiro 在企业级的项目开发中十分受欢迎，一般使用在单体服务中。但在微服务架构中，目前版本的 Apache Shiro是无能为力的Spring Security 来自 Spring Resource 社区，采用了注解的方式控制权限，熟悉Spring 的开发者很容易上手Spring Security。另外一个原因就是Spring Security易用与Spring boot工程，也容易集成到Spring Cloud构建的微服务系统中。

总结起来有以下几个特点：

  *     1. 代码耦合低
  *     2. 模块化
  *     3. 控制粒度细
  *     4. 熟悉spring容易上手
  *     5. spring boot 或者spring cloud的集成简单
  *     6. 庞大的社区与用户

Spring Security 和Spring Boot Security的关系如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e5a373afa7e5b5a0dd362f35a14f3a4.png)

## 3\. Security如何使用

### 3.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9e1d0e7ac3e1a1ac12f0ef35405efa19.png)

### 3.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1152e288fb9641dd2f0d1d2b8d25517.png)

### 3.3 security 配置

security需要自己写一个配置类，配置类集成于`WebSecurityConfigureAdapter`类
    
    
    @Configuration
    @EnableWebSecurity
    @EnableGlobalMethodSecurity(prePostEnabled = true)
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
        @Autowired
        private Environment environment;
    
        @Value("${web.security.user.name}")
        private String username;
    
        @Value("${web.security.user.pswd}")
        private String password;
    
        @Value("${web.security.user.role}")
        private String role;
    
        @Value("${web.security.admin.name}")
        private String adminName;
    
        @Value("${web.security.admin.pswd}")
        private String adminPswd;
    
        @Value("${web.security.admin.role}")
        private String adminRole;
    
        @Autowired
        public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
            auth.inMemoryAuthentication().withUser(username).password("{noop}" + password)
                    .roles(role.split(","));
            auth.inMemoryAuthentication().withUser(adminName).password("{noop}" + adminPswd)
                    .roles(adminRole.split(","));
    
        }
    
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .antMatchers("/css/**", "/index").permitAll()
                    .antMatchers("/user/**").hasRole("USER")
                    .antMatchers("/admin/**").hasRole("ADMIN")
                    .and()
                    .formLogin().loginPage("/login").failureUrl("/login-error")
                    .and()
                    .exceptionHandling().accessDeniedPage("/401")
                    .and()
                    .logout().logoutSuccessUrl("/");
        }
    
    }
    

这里的私有属性是在config.properties里面配置的用户名、密码与权限的信息。  
这里最好不要硬编码。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7214eb1de596456aed778527a313b579.png)

#### 3.3.1 configureGlobal方法

这个方法中，在内存中创建2个用户的信息，用户的用户名、密码以及密码的加密方式，和其具有的角色。  
密码的加密方式：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50997db3eca36f0a18367f08f489b479.png)  
这个方法里只有短短的两行代码，但是其完成了非常多的操作:

  *     1. 应用的每一个请求都要认证
  *     2. 自动生成了一个登陆表单
  *     3. 用指定的用户名密码进行认证
  *     4. 用户可以注销
  *     5. 阻止了CSRF的攻击
  *     6. Session Fixation的保护
  *     7. 安全Header
    * HTTP Strict Transport Security for secure requests
    * X-Content-Type_Options integration
    * Cache Control
    * X-XSS-Protection integration
    * XFrake-Option integration to help prevent Clickjacking
  *     8. 集成了如下方法:
    * HttpServletRequest#getRemoteUser()
    * HttpServletRequest.html#getUserPrincipal()
    * HttpServletRequest.html#isUserInRole(String)
    * HttpServletRequest.html#login(String,String)
    * HttpServletRequest.html#logout()

#### 3.3.2 启动登陆

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b1859ce4fdd265ba6509d8fb3b497773.png)  
其源码如下  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/08fd26d59afb1201273e5760b607de0e.png)  
写一个简单的html界面用来标识登陆成功。  
使用admin登陆  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/34b195e1307d7e0f4dadb274dc59d746.png)

#### 3.3.3 自定义配置 configure

| 代码                 | 配置内容                   |
|--------------------|------------------------|
| “/css/**”,"/index" | 不需要认证即可访问              |
| “/user/**”         | user目录下的界面需要验证user角色   |
| “/admin”           | admin目录下的界面需要验证admin角色 |
| formLogin          | 表单登陆界面是/login界面        |
| failureUrl         | 登陆失败的地址是/login-error   |
| exceptionHandling  | 异常会被重定向到401界面          |
| logout             | 支持注销                   |
| logoutSuccessUrl   | 注销后重定向到/               |

#### 3.3.4 controller

基于3.3.3的配置，实现controller
    
    
    @Controller
    public class MainController {
    
        @RequestMapping("/")
        public String root(){
            return "redirect:/index";
        }
    
        @RequestMapping("/index")
        public String index(){
            return "index";
        }
    
        @RequestMapping("/user/index")
        public String userIndex(){
            return "user/index";
        }
    
        @RequestMapping("/admin/index")
        public String adminIndex(){
            return "admin/index";
        }
    
        @RequestMapping("/login")
        public String login(){
            return "login";
        }
    
        @RequestMapping("/login-error")
        public String loginError(Model model){
            model.addAttribute("loginError",true);
            return "login";
        }
    
        @GetMapping("/401")
        public String accessDenied(){
            return "401";
        }
    }
    

#### 3.3.5 界面

login.html
    
    
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Login page</title>
        <base href="/">
        <link rel="stylesheet" href="/css/main.css" th:href="@{/css/main.css}" />
    </head>
    <body>
        <h1>Login page</h1>
        <p th:if="${loginError}" class="error">用户名或者密码错误!</p>
        <form th:action="@{/login}" method="post">
            <label for="username">用户名</label>:
            <input type="text" id="username" name="username" autofocus="autofocus"/><br/>
            <label for="password">密  码</label>:
            <input type="password" id="password" name="password" autofocus="autofocus"/><br/>
            <input type="submit" value="登录"/>
        </form>
        <p><a th:href="@{/index}">返回首页</a> </p>
    </body>
    </html>
    

index.html
    
    
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/xhtml"
          xmlns:th="http://www.thymeleaf.org"
          xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
    <head>
        <meta charset="UTF-8">
        <title>Hello Spring Boot Security for index</title>
        <base href="/">
        <link rel="stylesheet" href="css/main.css" th:href="@{/css/main.css}"/>
    </head>
    <body>
        <h1>Hello Spring Boot Security for index</h1>
        <p>这个界面没有受到保护.</p>
        <div th:fragment="logout" sec:authorize="isAuthenticated()">
            登录用户:<span sec:authentication="name"/>
            用户角色:<span sec:authentication="principal.authorities"/>
            <div>
                <form action="#" th:action="@{/logout}" method="post">
                    <input type="submit" value="登出" />
                </form>
            </div>
        </div>
        <ul>
            <li>点击<a href="/user/index" th:href="@{/user/index}">去/user/index被保护的界面</a> </li>
            <li>点击<a href="/admin/index" th:href="@{/admin/index}">去/admin/index被保护的界面</a> </li>
        </ul>
    </body>
    </html>
    

401.html
    
    
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/xhtml"
          xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
    <head>
        <meta charset="UTF-8">
        <title>401 Page</title>
    </head>
    <body>
        <div>
            <div>
                <h2>权限不够</h2>
            </div>
            <div sec:authorize="isAuthenticated()">
                <p>已有用户登录</p>
                <p>用户:<span sec:authentication="name" /></p>
                <p>角色:<span sec:authentication="principal.authorities"/></p>
            </div>
            <div sec:authorize="isAnonymous()">
                <p>未有用户登录</p>
            </div>
            <p>
                拒绝访问!
            </p>
        </div>
    </body>
    </html>
    

/user/index.html
    
    
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Hello Spring Security, User Index</title>
        <base href="/">
        <link rel="stylesheet" href="/css/main.css" th:href="@{/css/main.css}"/>
    </head>
    <body>
        <div th:substituteby="index::logout"/>
        <h1>这个界面是被保护界面,user角色可以访问</h1>
        <p><a href="/index" th:href="@{/index}">返回首页</a> </p>
        <p><a href="/admin" th:href="@{/admin/index}">去admin目录下的index</a> </p>
    </body>
    </html>
    

/admin/index.html
    
    
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Hello Spring Security, Admin Index</title>
        <base href="/">
        <link rel="stylesheet" href="/css/main.css" th:href="@{/css/main.css}"/>
    </head>
    <body>
    <div th:substituteby="index::logout"/>
    <h1>这个界面是被保护界面,admin角色可以访问</h1>
    <p><a href="/index" th:href="@{/index}">返回首页</a> </p>
    <p><a href="/admin" th:href="@{/user/index}">去user目录下的index</a> </p>
    </body>
    </html>
    

### 3.4 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd9e0b88711a9ca58253820e9e93d68f.png)  
不登陆访问/user或者admin的界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0aaffeed7abc662a7223c84289d303a9.png)  
访问admin的界面登陆user用户(user用户只有user角色)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59f5a5cdff68ea4166e223d256760fd6.png)  
相反的，访问user界面，登陆admin用户(admin用户有user和admin的角色)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69d45eb995bb9fd195fb1976c04bca46.png)  
访问admin下的界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5368fc967bbdf983fe9f0848c0706a3.png)  
然后登出  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d8ebeb14dab3440322eba7c219208414.png)  
登陆user角色访问user界面  
登陆失败  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fa4a748e9dfc2ffb29e02f17512041ac.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c0e3cbdc4f8ef698c8b1f34614872fc8.png)  
然后访问admin的界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d94970922dcc7445af1db658e8a0610.png)

## 4\. security 方法保护

### 4.1 创建实体
    
    
    public class Student {
    
        private String name;
    
        private int age;
    
        private String like;
    
        private Student(){
    
        }
    
        public static Student getBuild(){
            return new Student();
        }
    
        public Student name(String name){
            this.name = name;
            return Student.this;
        }
    
        public Student age(int age){
            this.age = age;
            return Student.this;
        }
    
        public Student like(String like){
            this.like = like;
            return Student.this;
        }
    
        public String getName(){
            return this.name;
        }
    
        public int getAge(){
            return this.age;
        }
    
        public String getLike(){
            return this.like;
        }
    
    }
    

### 4.2 创建服务

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a08e9e7559baa1822c5bcd37a7e90e4.png)

### 4.3 创建controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0cf9c47a5e71fcf8c1f6eca6df77d01e.png)

### 4.4 访问验证

可以看到，我们在service上有两个访问，一个是获取全部的学生的getStudentList的方法，这个方法只要有任意一个权限就能够访问。而另一个方法则必须拥有ADMIN的权限的用户登录才能进行访问。  
首先以USER权限进行登录：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5e0727cae6fb502e151ecf29398427f.png)  
然后获取所有的用户  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19ad85bcd6b2a29b00a62a548a8b77c3.png)  
然后进行尝试删除学生–小美  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e7f335314ef707d60d87f900613c190.png)  
发现其在controller接收到请求调用服务时，因权限不够而发生异常，但是我们之前在配置时配置，当有异常出现时，自动重定向到401的界面。  
所以，其展示的urlk地址是删除的地址，但是界面的内容确是，401的内容。  
接下来使用admin权限的用户进行登录，然后尝试删除学生。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ac6d6645c922d2d2fb7f1015a818426.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/afed6748edc7299afaab33258921f1fb.png)  
这里没有任何返回值，表示已经删除成功了，接下来重新获取所有的学生：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e764ab720dbecd838a6b4f6373d61ec5.png)

## 5\. 从数据库中读取用户认证信息

### 5.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fdb5e303e23c5df615b0a28f250dda1a.png)  
为了防止因为字符集的问题，需要手动增加依赖  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e1462aa0a8a290cc493ce6fd890294d.png)

### 5.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea9af0ba1ad190afca6b5c37ac9ba7a2.png)

### 5.3 创建实体

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69189c9e9ddf9cfed355007627ce200d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0634e77201d732b76fab2c162fc6855a.png)
    
    
    @Entity
    public class Subscriber implements UserDetails, Serializable {
    
        @Id
        @GeneratedValue(strategy = GenerationType.SEQUENCE)
        private Long id;
    
        @Column(nullable = false)
        private String password;
    
        @Column(nullable = false, unique = true)
        private String username;
    
        /*
         * OneToMany 是一对多的关系，关系由多的记录的属性维护(一般情况)
         * ManyToMany 是多对多的关系，关系由中间关系表维护(一般情况)
         */
    
        @ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
        @JoinTable(name = "subscriber_role", joinColumns = @JoinColumn(name = "subscriber_id", referencedColumnName = "id"),
        inverseJoinColumns = @JoinColumn(name = "role_id", referencedColumnName = "id"))
        /*
         * 这个JoinTable的大概含义是：
         * 这个中间关系由关系表维护，表名是 user_role
         * 关系表有两个字段，一个是 user_id,其映射的值是user表的id
         * 另一个是role_id,其映射的值是role表的id
         */
        private List<Role> authorities;
    
        public Subscriber(){
    
        }
    
        public Long getId(){
            return id;
        }
    
        public void setId(Long id){
            this.id = id;
        }
    
        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            return authorities;
        }
    
        public void setAuthorities(List<Role> authorities){
            this.authorities = authorities;
        }
    
        @Override
        public String getPassword() {
            return password;
        }
    
        public void setPassword(String password){
            this.password = password;
        }
    
        @Override
        public String getUsername() {
            return username;
        }
    
        public void setUsername(String username){
            this.username = username;
        }
    
        @Override
        public boolean isAccountNonExpired() {
            return true;
        }
    
        @Override
        public boolean isAccountNonLocked() {
            return true;
        }
    
        @Override
        public boolean isCredentialsNonExpired() {
            return true;
        }
    
        @Override
        public boolean isEnabled() {
            return true;
        }
    }
    
    

### 5.4 dao

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c9cf5ee18e58ca9cbd2403eb95908a1.png)

### 5.5 service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1755966eb94d787d0439a1e4d0cc790.png)

### 5.6 config

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f4cc9dd676ae40368cd28b14a6e6294.png)

### 5.7 启动验证

这是启动项目前的数据库中所有的表  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af04d8fc1f451abd26206af0ca62dde6.png)  
接着启动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/13cdba5ccb89b57e60630f67ef71e7a9.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cab6b11ec3235d5723cb88964341dee0.png)  
中间关系表与我们猜想的一致  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9a268727291661c37c141cb9f80080ee.png)  
不过有一点没有想到，这个中间关系表竟然有外键。

| 用户     | 权限    |
|--------|-------|
| userA  | USER  |
| userB  | USER  |
| userC  | USER  |
| adminA | ADMIN |
| adminB | ADMIN |
| adminC | ADMIN |
| allA   | USER  |
| allA   | ADMIN |
| allB   | USER  |
| allB   | ADMIN |
| allC   | USER  |
| allC   | ADMIN |

我们插入上述数据：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bdd27fef493220c8a156a43e90767ff3.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4cffbef836d6cda2a268b5ea745df132.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4fe8fad4e160aa64bd6b1dd87c452832.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/391f6c53f8a15f491942d17e8651acc2.png)  
接下来用这些用户尝试登陆，并且结合4中的逻辑，进行验证。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/990f8e01999e8b92bf40c968d8dae789.png)

改动点如上图所示  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09417c244254be203131a5cfa2f56af5.png)  
登陆  
userA  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d7a1ef56e020cf1d4ea69733ff2c1c49.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/220f0ae4ec4e9f9cb98116e888588b2f.png)  
adminA  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f84c6463b4ead44575109b50056466f6.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/268cbe9de842a230586eda1182bd6a5f.png)  
allA  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d83fa11d1489fa884073364c75067194.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b260271fc7a9bc8ee919f4d739cd429a.png)  
  
注意点：  
这里面有两个坑：  
1.password需要返回加密方式：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d2400ccb37b4160fdd30611075964eae.png)  
原因：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb2112f214b22d32de1a45faa439b17e.png)  
可选  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e4f4ea895297ec36349b76742435c2c.png)  
2.role返回的时候需要加前缀  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e19d93bb3200ff6379d2048e85e0418f.png)  
原因：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f064727981d29899d27de0585cb06871.png)  
使用配置的时候会自动加这个前缀，现在使用jpa则不会自动加前缀  

## 6\. 总结

使用Spring Security 还是比较简单的，没有想象中那么复杂。首先引入 Spring Security相关的依赖，然后写一个配置类，该配置类继承了 WebSecurityConfigurerAdapter，并在该配置类上加@EnableWebSecurity 注解开启 Web Security。再需要配置 AuthenticationManagerBuilder，AuthenticationManagerBuilder 配置了读取用户的认证信息的方式，可以从内存中读取，也可以  
从数据库中读取，或者用其他的方式。其次，需要配置 HttpSecurity，HttpSecurity 配置了请求的认证规则，即哪些 URI 请求需要认证、哪些不需要，以及需要拥有什么权限才能访问。最后，如果需要开启方法级别的安全配置，需要通过在配置类上加@EnableGlobalMethodSccuriy注解开启，方法级别上的安全控制支持secureEnabled、jsr250Enabled和 prePostEnabled这3种  
方式，用的最多的是prePostEnabled。其中，prePostEnabled 包括PreAuthorize和 PostAuthorize两种形式，一般只用到PreAuthorize这种方式。
