---
layout: post
title: SpringMVC利用AOP实现自定义注解记录日志
date: 2019-08-01 17:53:05
comments: true
tags: 
  - java
  - spring mvc
---
工作需要记录接口日志情况,默认spring mvc已配置可用
maven 添加jar
```linux
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>s
    <version>${servlet}</version>
    <scope>compile</scope>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjweaver</artifactId>
	<version>1.6.10</version>
</dependency>
<dependency>
	<groupId>aspectj</groupId>
	<artifactId>aspectjrt</artifactId>
	<version>1.5.3</version>
</dependency>
```
<!--more-->
spring-mvc.xml配置添加
```java
<!-- 最重要:::如果放在spring-context.xml中，这里的aop设置将不会生效 -->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```
自定义注解
<!--more-->
```java
package com.yougou.framework.logger.statistics;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @ClassName: LoggerStatistics 
 * @Description: 监听每个方法传入的接口、方法、参数个数、请求ip，打印日志
 * @author he.yh
 * @date 2017年5月11日 下午4:21:30
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LoggerStatistics {
	//方法注释
	String methodLog();
}

```
日志AOP类: LoggerStatisticsIntercepter
```java
package com.yougou.framework.logger.statistics;

import java.lang.reflect.Method;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import com.alibaba.dubbo.rpc.RpcContext;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @ClassName: LoggerStatisticsIntercepter 
 * @Description: 使用AOP对方法进行日志统计
 * @author he.yh
 * @date 2017年5月11日 下午4:21:10
 */
@Aspect
@Component
public class LoggerStatisticsIntercepter {
private static final Logger logger = LoggerFactory.getLogger("statisticsLog");
	
	@Around("@annotation(com.yougou.framework.logger.statistics.LoggerStatistics)")
	public Object doLoggerProfiler(final ProceedingJoinPoint joinPoint) throws Throwable {
		try {
			if(logger.isInfoEnabled()){
				StringBuffer sb = new StringBuffer(1000);
				String methodName=joinPoint.getSignature().getName();
				String apiName = joinPoint.getTarget().getClass().getInterfaces()[0].getName();
				String remoteHost = null;
				sb.append("api={},method={},paramNum={},remoteHost={},date={}");
				Object[] args = joinPoint.getArgs();
				try{
					RpcContext rpcContext = RpcContext.getContext();
					if(rpcContext!=null){
						remoteHost = rpcContext.getRemoteHost();
					}
				} catch (Exception e) {
					logger.error("RpcContext.getContext() Exception ",e);
				}
				 SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
				Object[] objects = {apiName,methodName,String.valueOf(args.length),remoteHost,df.format(new Date())};
				logger.info(sb.toString(), objects);
			}
		} catch (Exception e) {
			logger.error("LoggerStatisticsIntercepter Exception ",e);
		}
		return joinPoint.proceed();
	}
	
	protected Method getMethod(final JoinPoint joinPoint)
			throws NoSuchMethodException {
		final Signature sig = joinPoint.getSignature();
		if (!(sig instanceof MethodSignature)) {
			throw new NoSuchMethodException(
					"This annotation is only valid on a method.");
		}
		final MethodSignature msig = (MethodSignature) sig;
		final Object target = joinPoint.getTarget();
		return target.getClass().getMethod(msig.getName(),
				msig.getParameterTypes());
	}
}
```
接口使用案例
```java
package com.yougou.ordercenter.api.asm.impl;

import com.yougou.framework.logger.Logger;
import com.yougou.framework.logger.LoggerFactory;
import com.yougou.framework.logger.statistics.LoggerStatistics;
import com.yougou.framework.logger.statistics.LoggerStatisticsConstant;

@Component("orderForSaleApi")
public class OrderForSaleApiImpl implements IOrderForSaleApi {

	protected Logger logger = LoggerFactory.getLogger(this.getClass());
	
	@Override
	@LoggerStatistics(methodLog = "接口调用统计日志")
	public String getSaleApplyRefundNo(String saleApplyNo) {
		return null;
	}
}
```
这样每调用IOrderForSaleApi.getSaleApplyRefundNo方法就会打印出一条日志,如下:
```file
[INFO ] 2017-05-11 16:59:38,642 api=com.yougou.ordercenter.api.invoice.IInvoiceNewApi,
method=updateInvoiceForOMSReSplitOrder,paramNum=4,
remoteHost=10.0.20.107,date=2017-05-11 16:59:38 
```
日志打印内容和格式可以根据LoggerStatisticsIntercepter.doLoggerProfiler上根据需求调整
注解补充:
　@Before – 目标方法执行前执行
　@After – 目标方法执行后执行
　@AfterReturning – 目标方法返回后执行，如果发生异常不执行
　@AfterThrowing – 异常时执行
　@Around – 在执行上面其他操作的同时也执行这个方法