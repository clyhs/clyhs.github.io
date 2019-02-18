---
title: shiro redis session共享
date: 2019-02-16 10:56:57
category: shiro
tags: [shiro,redis]
---

### 引入依赖包

在pom文件引入如下

```
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
	<version>2.9.0</version>
</dependency>

<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-redis</artifactId>
	<version>1.6.0.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session</artifactId>
	<version>1.0.2.RELEASE</version>
</dependency>

<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-core</artifactId>
	<version>1.2.3</version>
</dependency>
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-web</artifactId>
	<version>1.2.3</version>
</dependency>
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring</artifactId>
	<version>1.2.3</version>
</dependency>
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>3.2.2</version>
</dependency>
```

### 配置spring-context.xml

```
<!--单机版-->
<!--
<bean id="redisManager" class="org.crazycake.shiro.RedisManager">
    <property name="host" value="192.168.0.7:6379"/>
    <property name="password" value="123456" />
</bean>
-->

<!--分布式-->
<!--
<bean id="redisManager" class="org.crazycake.shiro.RedisClusterManager">
    <property name="host" value="192.168.0.16:8001,192.168.0.16:8002,192.168.0.16:8003,192.168.0.7:8004,192.168.0.7:8005,192.168.0.7:8006"/>
    <property name="password" value="123456" />
</bean>
-->

<!--高可用-->
<bean id="redisManager" class="org.crazycake.shiro.RedisSentinelManager">
    <property name="host" value="192.168.0.7:26000,192.168.0.16:26001,192.168.0.16:26002"/>
    <!--  <property name="password" value="123456" />-->
</bean>
    
<bean id="redisCacheManager" class="org.crazycake.shiro.RedisCacheManager">
	<property name="redisManager" ref="redisManager" />
	<!-- optional properties
	<property name="expire" value="1800"/>
	<property name="keyPrefix" value="shiro:cache:" />
	<property name="principalIdFieldName" value="id" />
	-->
</bean>

<bean id="credentialsMatcher" class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
	<property name="hashAlgorithmName" value="md5" />
	<property name="hashIterations" value="2" />
	<property name="storedCredentialsHexEncoded" value="true" />
</bean>
<bean id="customRealm" class="com.pas.cloud.module.shiro.CustomRealm">
	<property name="cachingEnabled" value="false" />
	<property name="authenticationCachingEnabled" value="false" />
	<!-- 
	<property name="credentialsMatcher" ref="credentialsMatcher" />
	<property name="cachingEnabled" value="true" />
	<property name="authenticationCachingEnabled" value="true" />
	<property name="authenticationCacheName" value="authenticationCache" />
	<property name="authorizationCachingEnabled" value="true" />
	<property name="authorizationCacheName" value="authorizationCache" /> -->
</bean>

<bean id="redisSessionDAO" class="org.crazycake.shiro.RedisSessionDAO">
	<property name="redisManager" ref="redisManager" />
	<!-- optional properties
	<property name="expire" value="-2"/>
	<property name="keyPrefix" value="shiro:session:" />
	-->
</bean>
<!-- sessionIdCookie的实现,用于重写覆盖容器默认的JSESSIONID -->
<bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
	<!-- cookie的name,对应的默认是 JSESSIONID -->
	<constructor-arg name="name" value="SHAREJSESSIONID" />
	<!-- jsessionId的path为 / 用于多个系统共享jsessionId -->
	<property name="path" value="/" />
	<property name="httpOnly" value="true" />
</bean>
<!-- sessionManager -->
<!-- session管理器 -->
<bean id="sessionManager"
	class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
	<!-- 设置全局会话超时时间，默认30分钟(1800000) -->
	<property name="globalSessionTimeout" value="1800000" />
	<!-- 是否在会话过期后会调用SessionDAO的delete方法删除会话 默认true -->
	<property name="deleteInvalidSessions" value="true" />

	<!-- 会话验证器调度时间 -->
	<property name="sessionValidationInterval" value="1800000" />

	<!-- session存储的实现 -->
	<property name="sessionDAO" ref="redisSessionDAO" />
	<!-- sessionIdCookie的实现,用于重写覆盖容器默认的JSESSIONID -->
	<property name="sessionIdCookie" ref="sessionIdCookie" />
	<!-- 定时检查失效的session -->
	<property name="sessionValidationSchedulerEnabled" value="true" />
</bean>

<!-- securityManager安全管理器 -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	<property name="realm" ref="customRealm" />
	<!-- ehcahe缓存shiro自带 <property name="cacheManager" ref="shiroEhcacheManager"></property> -->

	<!-- redis缓存 -->
	<property name="cacheManager" ref="redisCacheManager" />

	<!-- sessionManager -->
	<property name="sessionManager" ref="sessionManager"></property>
</bean>

<bean
class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
	<property name="staticMethod"
			value="org.apache.shiro.SecurityUtils.setSecurityManager" />
	<property name="arguments" ref="securityManager" />
</bean>

<!-- 记住我cookie -->
<bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
	<!-- rememberMe是cookie的名字 -->
	<constructor-arg value="rememberMe" />
	<!-- 记住我cookie生效时间30天 -->
	<property name="maxAge" value="2592000" />
</bean>

<!-- rememberMeManager管理器，写cookie，取出cookie生成用户信息 -->
<bean id="rememberMeManager" class="org.apache.shiro.web.mgt.CookieRememberMeManager">
	<property name="cookie" ref="rememberMeCookie" />
</bean>


<bean id="shiroManager" class="com.pas.cloud.module.shiro.ShiroManagerImpl">
	<property name="xtcdService" ref="xtcdService"></property>
</bean>

<bean id="logoutFilter" class="org.apache.shiro.web.filter.authc.LogoutFilter">
	<property name="redirectUrl" value="/system/loginPage.html" />
</bean>

<!-- 配置shiro的过滤器工厂类，id- shiroFilter要和我们在web.xml中配置的过滤器一致 -->
<!--  
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">-->
<bean id="shiroFilter" class="com.pas.cloud.module.shiro.PasShiroFilterFactoryBean">
	<property name="securityManager" ref="securityManager" />
	<property name="loginUrl" value="/system/loginPage.html" />
	<property name="successUrl" value="/system/index.html" />
	<property name="unauthorizedUrl" value="/unauth.jsp" />
	<property name="filterChainDefinitions" value="#{shiroManager.loadFilterChainDefinitions()}" />
<!-- <property name="filterChainDefinitions"> <value> /system/loginPage= 
anon /system/login = anon /system/loginout= logout /system/index=authc,perms[system:index] 
/module/user/add =authc,perms[module:user:add] /module/user/index = authc,perms[module:user:index] 
/module/user/del =authc,perms[module:user:del] /module/xtcd/index = authc, 
perms[module:xtcd:index] /module/** = authc </value> </property> -->
	<property name="filters">
		<map>
			<entry key="logout" value-ref="logoutFilter" />
		</map>
	</property>
</bean>


<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
```

