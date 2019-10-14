---
title: 个人笔记--struts2对Action的权限拦截
date: 2016-10-24 10:10:45
categories:
- 后端
tags:
- java
- javaee
---

## 编写一个类实现com.opensymphony.xwork2.interceptor.Interceptor接口

PermissionInterceptor.java

```java
package cn.sky.bookshop.interceptor;

import org.apache.struts2.ServletActionContext;
import cn.sky.bookshop.utils.DateUtil;
import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.interceptor.Interceptor;

public class PermissionInterceptor implements Interceptor {

    private static final long serialVersionUID = -1908127830415801520L;

    private String includeUrl;

    public void destroy() {
    }

    public void init() {
    }

    public String intercept(ActionInvocation invocation) throws Exception {
	//会话session中取出uname
	Object obj = ServletActionContext.getRequest().getSession().getAttribute("uname");
	//获取访问路径
	String path = ServletActionContext.getRequest().getServletPath();
	//如果存在uname或者访问路径包含在includeUrl中
	if (obj != null || includeUrl.contains(path)) {
	    return invocation.invoke(); //继续调用下一个拦截器，如果没有则执行Action
	}
	ServletActionContext.getRequest().setAttribute("errorInfo","No permission to operate the page");//保存错误信息
	return "disallow";//返回视图
    }

    public String getIncludeUrl() {
	return includeUrl;
    }

    public void setIncludeUrl(String includeUrl) {
	this.includeUrl = includeUrl;
    }
}
```

## 配置struts.xml 文件
1	定义一个自己的默认包
```xml
<package name="my-struts-default" namespace="/" extends="struts-default">

		<interceptors>
			<!-- 这里是定义一个拦截器 -->
			<interceptor name="permission" class="cn.sky.bookshop.interceptor.PermissionInterceptor">
					<!-- 拦截器的初始化注入参数 -->
					<param name="includeUrl">/user/user_login.action,/getcode</param>
			</interceptor>
			<!-- 这里定义一个拦截器栈 -->
			<interceptor-stack name="myDefalutStack">
				<interceptor-ref name="defaultStack"></interceptor-ref>
				<interceptor-ref name="permission"></interceptor-ref>
			</interceptor-stack>
		</interceptors>
		<!--设置默认的拦截器栈(访问Action前默认调用)-->
	<default-interceptor-ref name="myDefalutStack"></default-interceptor-ref>
		<!-- 全局result -->
		<global-results>
			<result name="disallow" type="redirect">/login.jsp</result>
		</global-results>
		<action name="getcode" class="cn.sky.bookshop.action.VerifyCodeAction">
		</action>

	</package>
```
2	以后的包都继承上面my-struts-default

## 注意

这里只能实现对访问Action时实现拦截，对jsp的访问不可能实现拦截。(可以将自定义拦截器栈中的默认拦截器
defaultStack注释掉，访问jsp发现根本没执行过此拦截器。可想而知，在拦截器工作之前对jsp的页面请求已经做出响应了。)