---
title: Servlet之过滤器实现权限拦截
date: 2016-10-24 10:10:45
categories:
- 后端
tags:
- java
- javaee
---

## 编写一个Java类实现javax.servlet.Filter接口

```java
package cn.edu.sxu.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class PermissionFilter implements Filter {
    private String includeUrl;
    public void destroy() {
	System.out.println("权限拦截销毁...");
    }

    public void doFilter(ServletRequest request, ServletResponse response,
	    FilterChain chain) throws IOException, ServletException {
	HttpServletRequest req = (HttpServletRequest)request;
	HttpServletResponse resp = (HttpServletResponse)response;
	//从会话中拿登录后session保存的uname
	Object obj = req.getSession().getAttribute("uname");
	//获取请求路径
	String path = req.getServletPath();
	//如果会话中保存了uname或者访问路径在includeUrl中
	if(null!=obj||includeUrl.contains(path))
	{
	    chain.doFilter(req,resp);
	}
	else
	{
	    System.out.println("重定向 ...");
	    resp.sendRedirect(req.getContextPath()+"/login.jsp");
	}
    }

    public void init(FilterConfig config) throws ServletException {
	System.out.println("权限拦截启用...");
	//从web.xml中PermissionFilter加载参数获取可访问路径
	this.includeUrl = config.getInitParameter("includeUrl");
    }

}
```

## 在web.xml中配置此过滤器

```xml
<filter>
	<filter-name>PermissionFilter</filter-name>
	<filter-class>cn.edu.sxu.filter.PermissionFilter</filter-class>
	<init-param>
		<param-name>includeUrl</param-name>  
                <!--这里是允许通过的访问路径-->
		<param-value>/index.jsp,/login.jsp,/register.jsp</param-value>
	</init-param>
</filter>

<filter-mapping>
	<filter-name>PermissionFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```
以上就可以实现权限拦截功能了
## 与Struts2整合

下面将此过滤器与struts2中StrutsPrepareAndExecuteFilter过滤器整合代码 
```java
package cn.sky.bookshop.filter;

import java.io.IOException;

import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts2.dispatcher.Dispatcher;
import org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter;

import cn.sky.bookshop.utils.DateUtil;

public class StrutsExtendsI18nFilter extends StrutsPrepareAndExecuteFilter {
    private String includeUrl;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
	this.includeUrl = filterConfig.getInitParameter("includeUrl");
	super.init(filterConfig); // 调用父类(struts2核心过滤器的初始化方法)初始化方法初始化
    }

    @Override
    protected void postInit(Dispatcher dispatcher, FilterConfig filterConfig) {
	System.out.println("这里你可以让struts2初始化再做些什么事。。。");
	super.postInit(dispatcher, filterConfig); // 父类这个方法是空的,这句话是废话
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
	    FilterChain chain) throws IOException, ServletException {
	// 使用国际化封装request,重写getLocale()方法
	HttpServletRequest myrequest = new MyHttpRequest((HttpServletRequest) request);
	HttpServletResponse resp = (HttpServletResponse) response;
	//获取会话session里的uname
	Object obj = myrequest.getSession().getAttribute("uname");
	//获取访问路径
	String path = myrequest.getServletPath();
	//如果session中存在uname,或includeUrl中包含了访问路径path
	if (null != obj || includeUrl.contains(path)) {
	    // 调用父类Struts2核心过滤器的doFilter方法
	    super.doFilter(myrequest, response, chain);
	} else {
	    // 重定向回登录页面
	    resp.sendRedirect(myrequest.getContextPath() + "/login.jsp");
	}
	// super.doFilter(myrequest, response, chain);
    }

    @Override
    public void destroy() {
    }

}
```
## 在web.xml中的配置

```xml
  <filter>
  	<filter-name>struts2.3</filter-name>
  	<filter-class>cn.sky.bookshop.filter.StrutsExtendsI18nFilter</filter-class>
  	<init-param>
  		<param-name>includeUrl</param-name>
  		<param-value>/login.jsp,/register.jsp,/index.jsp,/user/user_login.action</param-value>
  	</init-param>
  </filter>
  <filter-mapping>
  	<filter-name>struts2.3</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
```