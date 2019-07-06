---
title: SpringSecurity控制授权(鉴权)功能介绍
categories:
 - Spring-Security
tags:
 - 后端开发
description: spring security中的除了用户登录校验相关的过滤器,最后还包含了鉴权功能的过滤器,还有匿名资源访问的过滤器链,相关的图解如下...
---

#### 1.spring security 过滤器链

​	spring security中的除了用户登录校验相关的过滤器,最后还包含了鉴权功能的过滤器,还有匿名资源访问的过滤器链,相关的图解如下:
![VWDIYQ.png](https://s2.ax1x.com/2019/06/12/VWDIYQ.png)

#### 2.控制授权的相关类

​	这里是整个spring security的过滤器链中的授权流程中控制权限的类的相关图示:

![VWDzY4.png](https://s2.ax1x.com/2019/06/12/VWDzY4.png)

​	这里主要是从AccessDecisionVoter的投票者(译称)把信息传递给投票管理者AccessDecisionManager,最终来判断是过还是不过(也就是有没有权限).有两种可能的类:

​	1.不管有多少请求投票不过,只要有一个过就可以通过(UnanimousBased);

​	2.不管有多少请求投票通过,只要有一个不通过就不让通过(AffirmativeBased);

​	3.比较投通过和不通过的个数,谁多久就按照谁的方式来(Consensusbased).

​	这里可以可能听起来有点绕,但实际上就是三种控制权限的方式类,我们可以认为Spring security已经帮我们做好了最终的判断,我们只需要当一个旁观者即可.

​	我们再来关注SecurityContextHolder这个类,他会将我们的权限信息封装到Authentication中,SecurityConfig则是相关的Spring security的配置信息,这个类会将相关的信息传递到ConfigAttribute中.

#### 3.配置简单的权限

​	这个在身份信息固定,并且不会经常变动的情况下可以按照如下配置,否则不建议这么做,这里只适用于简单的场景.

MyUserDetailsService:

```java
private SocialUserDetails buildUser(String userId) {
		// 根据用户名查找用户信息
		//根据查找到的用户信息判断用户是否被冻结
		/**
		 * 可以从数据库查出来用户名和密码进行比对,为了方便我这里就直接固定了
		 */
		String password = passwordEncoder.encode("123456");
		logger.info("数据库密码是:"+password);
		//注意这里配置角色的时候需要加ROLE_前缀
		return new SocialUser(userId, password,
				true, true, true, true,
				AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_ADMIN"));
	}
```

BrowserSecurityConfig:

```java
//这里是硬编码权限 只限于简单的用户权限 这里的角色名称严格区分大小写
   //这里可以指定HttpMethod  如HttpMethod.GET,
    .antMatchers("/user/**").hasRole("ADMIN")
        .anyRequest()     //所有请求
        .authenticated() //都需身份认证
```

#### 4.权限表达式

​	之前我们一直都有用到权限表达式,比如最常用的permitAll和Authenticated,这个权限表达式就是允许所有都能访问的意思,其他的相关的权限表达式如下所示:

![VWrSfJ.png](https://s2.ax1x.com/2019/06/12/VWrSfJ.png)

这里可以改写权限的表达如下:

```java
.antMatchers("/user/**").access("hasRole('ADMIN') and hasIpAddress('XXXXXX')")
        .anyRequest()     //所有请求
        .authenticated() //都需身份认证
```

#### 5.spring security控制授权代码封装

![VWriOx.png](https://s2.ax1x.com/2019/06/13/VWriOx.png)

代码相关:

AuthorizeConfigProvider:

```java
/**
 * 这个是授权的类
 */
public interface AuthorizeConfigProvider {

    void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config);
}
```

CityAuthorizeConfigProvider:

```java
//配置permitAll的路径
@Component
public class CityAuthorizeConfigProvider implements AuthorizeConfigProvider {
    @Autowired
    private SecurityProperties securityProperties;
    @Override
    public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
        config.antMatchers(
                "/static/**","/page/login","/page/failure","/page/mobilePage",
                "/code/image","/code/sms","/authentication/mobile",securityProperties.getBrower().getSignUPUrl(),
                "/user/register","/page/registerPage","/page/invalidSession","/page/logoutSuccess",securityProperties.getBrower().getSignOutUrl()

        )
                .permitAll();
    }
}
```

AuthorizeConfigManager:

```java
/**
 * AuthorizeConfigManager
 */
public interface AuthorizeConfigManager {
    void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config);
}
```

CityAuthorizeConfigManager:

```java
@Component
public class CityAuthorizeConfigManager implements AuthorizeConfigManager  {

    @Autowired
    private Set<AuthorizeConfigProvider> authorizeConfigProviders;

    @Override
    public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
        for (AuthorizeConfigProvider authorizeConfigProvider : authorizeConfigProviders) {
            authorizeConfigProvider.config(config);
        }
        //除了自己配置的CityAuthorizeConfigProvider的内容外 其他的都需要认证
        config.anyRequest().authenticated();
    }
}
```

在user中我们可以配置访问的页面权限:

DemoAuthorizeConifgProvider:

```java
@Component
public class DemoAuthorizeConifgProvider implements AuthorizeConfigProvider {

    @Override
    public void config(ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {

        config.antMatchers("/demo.html","/user/**")
                .hasRole("ADMIN");
    }
}
```

注意在BrowserSecurityConfig中注释掉相关的代码后需要加入如下代码:

```java
@Autowired
private AuthorizeConfigManager authorizeConfigManager;

在configure方法中调用:
authorizeConfigManager.config(http.authorizeRequests());
```

配置完后我们登陆进来,访问http://localhost:8060/demo.html 显示403权限拒绝,这个就说明权限生效了

#### 4.项目git地址

(喜欢记得点星支持哦,谢谢!)

[https://github.com/fengcharly/spring-security-oauth2.0](https://github.com/fengcharly/spring-security-oauth2.0)