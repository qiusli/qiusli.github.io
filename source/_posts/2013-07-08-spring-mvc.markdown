---
layout: post
title: "Spring MVC学习笔记"
date: 2015-10-15 22:55:12
categories: Technology
---
学习Spring很长时间了，但是一直也没有认真地总结一次，总是陷入学习了忘记的怪圈（其实也不是怪圈，就是学习了没有总结）。 今天我就写一点东西来总结我的Spring MVC学习之旅。 这个得从我的Sponsor给我布置的家庭作业讲起，在这里面我学会了很多Spring的知识。

可以假设Spring是一个大的容器，里面放着各种各样的网页、文件等以供一个个的request访问。所以，我想从web.xml的配置说起。
<!-- more -->
``` xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>    
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext-servlet.xml
        classpath:applicationContextDataSource.xml
    </param-value>
</context-param>    
<servlet>
    <servlet-name>bankingSystem2</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>bankingSystem2</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
上面的配置的意思是：servlet-mapping配置了需要被捕获的请求，url-pattern是匹配request中跟的那个地址，例如’/abc’。这里只是一个斜杠，意思是捕获所有的请求，捕获这个请求的servlet名字叫bankingSystem2。 在捕获一个请求后，Spring会根据servlet-name中的名字在这个xml中找到匹配的一个servlet，然后servlet-class是这个servlet文件的所在地。这里的DispatcherServlet是Spring的类，所有发往由Spring 容器管理的资源的请求都由它统一收集，然后它会求助Spring中内置的HandlerMapping已决定该请求被哪个controller处理。最后根据不同的请求目的地再转发给不同的controller，这个后面会讲到。这里有一点值得注意，我们在web.xml中定义了一个名字叫bankingSystem2的servlet，那么就需要在同一个文件夹下面定义一个名叫bankingSystem2-servlet.xml这样一个配置文Spring会默认到里面去读取配置和加载bean，如果没有会报错。

ContextLoaderListener的作用就是启动Web容器时，自动装配ApplicationContext的配置信息。因为它实现了ServletContextListener这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。 在上面的contextConfigLocation里面配置了想要在Spring容器启动的时候加载的bean。 看完了上述的web.xml，我们进入在那个配置文件中配置的applicationContext-servlet.xml中一探究竟。
``` xml
<context:component-scan base-package="bank.icbc.controller"/>    
<context:component-scan base-package="bank.icbc.domain"/>    
<mvc:annotation-driven/>    
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
<property name="prefix" value="/WEB-INF/jsp/"/>
<property name="suffix" value=".jsp"/>
</bean>
```  
首先，在这个xml配置文件中定义了一个bean，里面存放着一个resolver，他是Spring中负责显示页面的。即controller处理完一个request后，会返回一个string告诉Spring现在该显示哪个页面了，然后Spring就找到InternalResourceViewResolver，让它告诉该到哪去找到相应的页面来显示。例如，如果一个controller返回字符串abc，那么根据上面的配置，在/WEB-INF/jsp/abc.jsp这个页面就应该被显示。 上面还有一个context:component-scan，它的作用是‘扫描’相应的package，把这些package中的所有类纳入Spring的管理范围来，这样的好处是，如果相应包中有autowire bean中的类，只有主类在Spring的管理范围， 被引用的bean才能被成功地autowire。

同时，有些类（例如服务类）会在类上面表示诸如@Service这样的annotation，它被扫描到后，也会被Spring纳入到管理，这样其他类也能autowire它了。 annotation-driven：表示支持annotation，不写的话所有的annotation注解都无效。

在说完配置后，我们来模拟一个请求，来看看Spring在这个过程中的运行过程。首先，我们来到系统的初始页面：
``` xml
Welcome to the Banking System! <br/>  <c:url value="/addCustomer" var="addCustomer"/>  
1. <a href="${addCustomer}">add Customer</a>  <c:url value="/withdraw" var="withdraw"/>  
2. <a href="${withdraw}">withdraw</a>  <c:url value="/deposit" var="deposit"/>  
3. <a href="${deposit}">deposit</a>  
``` 
在这个页面中，我们定义了3个URL，请求的地址分别是/addCustomer, /withdraw和/deposit。这时就需要有相应的controller来捕获对这些地址的请求，请看下面：
``` java
@Controllerpublic
class CustomerController {
    @Autowired
    @Qualifier("bank")
    private Bank bank;
    @Autowired
    @Qualifier("customer")
    private Customer customer;

    @RequestMapping(value = "/addCustomer", method = RequestMethod.GET)
    public String addCustomer(@ModelAttribute Customer customer) {
        return "AddCustomer";
    }

    @RequestMapping(value = "/addCustomer", method = RequestMethod.POST)
    public String saveCustomer(@ModelAttribute Customer customer, ModelMap modelMap) {
        bank.addCustomer(customer);
        Customer theCustomer = bank.getCustomer(customer.getNickname());
        modelMap.addAttribute("nickname", theCustomer.getNickname());
        modelMap.addAttribute("dateOfBirth", theCustomer.getDateOfBirth());
        modelMap.addAttribute("emailAddress", theCustomer.getEmailAddress());
        return "ShowCustomer";
    }

    @RequestMapping(value = "/openAccount", method = RequestMethod.GET)
    public String openAccount(@ModelAttribute Account account, ModelMap modelMap) {
        bank.addAccount(customer);
        Account account1 = bank.getAccount(customer.getNickname());
        modelMap.addAttribute("nickname", account1.getBalance());
        modelMap.addAttribute("balance", account1.getBalance());
        modelMap.addAttribute("joinDate", account1.getJoinDate());
        modelMap.addAttribute("isPremium", account1.isPremium());
        return "ShowAccount";
    }

    @RequestMapping(value = "/withdraw", method = RequestMethod.GET)
    public String goToWithdrawPage(@ModelAttribute Customer customer) {
        return "Withdraw";
    }

    @RequestMapping(value = "/deposit", method = RequestMethod.GET)
    public String goToDepositPage(@ModelAttribute Customer customer) {
        return "Deposit";
    }

    @RequestMapping(value = "/deposit", method = RequestMethod.POST)
    public String deposit(final HttpServletRequest request, ModelMap modelMap) {
        String nickname = request.getParameter("nickname");
        Account account = bank.getAccount(nickname);
        account.deposit(Integer.parseInt(request.getParameter("balance")), nickname);
        Account account1 = bank.getAccount(nickname);
        modelMap.addAttribute("balance", account1.getBalance());
        return "ShowBalance";
    }

    @RequestMapping(value = "/withdraw", method = RequestMethod.POST)
    public String withdraw(final HttpServletRequest request, ModelMap modelMap) {
        String nickname = request.getParameter("nickname");
        Account account = bank.getAccount(nickname);
        account.withdraw(account.getBalance(), nickname);
        Account account1 = bank.getAccount(nickname);
        modelMap.addAttribute("balance", account1.getBalance());
        return "ShowBalance";
    }

    @RequestMapping(value = "/welcome")
    public String backToWelcomePage() {
        return "Welcome";
    }

    @ExceptionHandler(Exception.class)
    public String handleException() {
        return "Exception";
    }
}
```
这个类是一个controller（因为被标注了@Controller 这个注解）。然后，它会捕获上面那个jsp页面发出来的请求，而具体哪个方法捕获对哪一个URL的请求，请看具体方法上面标示的@RequestMapping 注解。它规定了handle的请求的类型和URL。在处理请求的controller方法的签名中，我们看到request和modelmap。它们都被Spring容器管理，所以直接在方法参数中加进来就能直接使用它们了。ModelMap 的左右是存放一些值，用于JSP页面显示。

这就是一次大致的Spring MVC 例子的讲解。