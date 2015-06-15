# spring-boot-actuator-monitor
基于spring boot的 监控平台

#spring-boot-admin 监控后台



1.   新建一个maven工程 boot-admin ， 继承 spring-boot-starter-parent
	

		<parent>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-parent</artifactId>
	        <version>1.2.4.RELEASE</version>
	    </parent>

2.	 加入依赖


		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-server</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-server-ui</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>com.googlecode.json-simple</groupId>
            <artifactId>json-simple</artifactId>
            <optional>false</optional>
        </dependency>

3.	 加入plugin 以 exec jar包运行


			<plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                        <configuration>
                            <classifier>exec</classifier>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

4.   新建执行执行类   com.izerui.boot.admin.Application.java


		@SpringBootApplication
		@EnableAdminServer
		@EnableDiscoveryClient
		public class Application {
		
		    public static void main(String[] args) {
		        SpringApplication.run(Application.class,args);
		    }
		}

5.   声明监控后台的应用名称、端口、上下文、基本授权验证用户名、密码等信息  application.properties


		# application
		server.port = 8888
		
		# basic security
		security.ignored=/api/**
		security.user.name=boot
		security.user.password=boot1234


6.  mvn install 后 执行 java -jar boot-admin-exec.jar

7.  访问 http://主机ID:端口/boot-admin



----------


# 注册应用到监控后台

1.   确保应用继承 spring-boot-starter-parent


		<parent>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-parent</artifactId>
	        <version>1.2.4.RELEASE</version>
	    </parent>

2.   引入依赖


		<dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>com.googlecode.json-simple</groupId>
            <artifactId>json-simple</artifactId>
            <optional>false</optional>
        </dependency>

3.   配置被监控的应用的名称和监控后台地址 application.properties

		> jar 方式运行

		# web application
		server.port=28584
		info.version=@pom.version@
		server.context-path=/@pom.artifactId@
		endpoints.jmx.domain=@pom.artifactId@
		spring.application.name=@pom.artifactId@

		# boot admin
		spring.boot.admin.url=http://192.168.1.128:8888/boot-admin

		---------------------------------------------------

		> war 方式运行
		
		# web application
		endpoints.jmx.domain=@pom.artifactId@
		spring.application.name=@pom.artifactId@
		
		# boot admin
		spring.boot.admin.url=http://192.168.1.128:8888
		spring.boot.admin.client.service-url=http://192.168.1.106:8098/qq-pay-web



		两种方式记得别忘了添加logging配置
		# LOGGING
		logging.file=/tmp/logs/offline-proxy-tcp.log
		logging.level.*= INFO


4.   配置logback的jmx接口调用

		<?xml version="1.0" encoding="UTF-8"?>
		<configuration>
		    <include resource="org/springframework/boot/logging/logback/base.xml"/>
		    <jmxConfigurator/>
		    <logger name="com.izerui" level="DEBUG"/>
		    <root level="INFO">
		        <appender-ref ref="CONSOLE" />
		        <appender-ref ref="FILE" />
		    </root>
		</configuration>

5.   发布自定义jmx接口


		在spring bean 类的头信息上增加注解 @ManagedResource(currencyTimeLimit = 20) 和 @Lazy

		声明属性： @ManagedAttribute(description = "当前TCP连接数")

		声明方法： @ManagedOperation(description = "关闭对应IP的所有连接")
			      @ManagedOperationParameters(
			            @ManagedOperationParameter(name = "ip", description = "终端IP")
			      )



参考资料：

* https://github.com/codecentric/spring-boot-admin.git

* http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-jmx.html

* https://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator

* http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#_auto_configured_healthindicators
