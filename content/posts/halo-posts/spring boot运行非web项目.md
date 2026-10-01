---
title: spring boot运行非web项目
id: 68
date: 2024-09-27 19:10:52
auther: admin
cover: 
excerpt: 前言一直以来，一直使用springboot来创建基于web的项目，今天突然想到，springboot是否可以用于非web项目的使用。非web运行办法启动类main方法后面执行@SpringBootApplicationpublic class StudyLockApp {    public sta
permalink: /?p=68
categories:
 - spring
tags: 
 - java
---

## 前言

一直以来，一直使用springboot来创建基于web的项目，今天突然想到，springboot是否可以用于非web项目的使用。

## 非web运行办法

### 启动类main方法后面执行

```java
@SpringBootApplication
public class StudyLockApp {
    public static void main(String[] args) {
        ConfigurableApplicationContext configurableApplicationContext = SpringApplication.run(StudyLockApp.class, args);
        RedisService redisService = configurableApplicationContext.getBean(RedisService.class);
        Boolean lock = redisService.getLock("lock", 1);
        System.out.println(lock);
    }

}
```

- 通过SpringApplication.run拿到bean容器
- 获取到已经注入进去的业务bean
- 执行相应的方法

### 实现ApplicationRunner接口

```java
@SpringBootApplication
public class StudyLockApp implements ApplicationRunner, ApplicationContextAware {
    private ApplicationContext applicationContext;

    public static void main(String[] args) {
        SpringApplication.run(StudyLockApp.class, args);
    }


    @Override
    public void run(ApplicationArguments args) throws Exception {
        RedisService redisService = applicationContext.getBean(RedisService.class);
        for(int i = 0;i<10;i++){
            Thread thread = new Thread(new ConsumerThread(i,redisService));
            thread.start();
        }
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

### 实现CommandLineRunner接口

```java
@SpringBootApplication
public class StudyLockApp implements CommandLineRunner, ApplicationContextAware {
    private ApplicationContext applicationContext;

    public static void main(String[] args) {
        SpringApplication.run(StudyLockApp.class, args);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void run(String... args) throws Exception {
        RedisService redisService = applicationContext.getBean(RedisService.class);
        for(int i = 0;i<10;i++){
            Thread thread = new Thread(new ConsumerThread(i,redisService));
            thread.start();
        }
    }
}

```

### Springboot源码

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
		runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
		AnnotationAwareOrderComparator.sort(runners);
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}

	private void callRunner(ApplicationRunner runner, ApplicationArguments args) {
		try {
			(runner).run(args);
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute ApplicationRunner", ex);
		}
	}

	private void callRunner(CommandLineRunner runner, ApplicationArguments args) {
		try {
			(runner).run(args.getSourceArgs());
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute CommandLineRunner", ex);
		}
	}
```

其实就是在执行容器初始化后再调用实现的run方法而已。