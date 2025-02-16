---
layout: post
title:  IntellJ에서 SpringBoot 사용 시 ClassLoader Error 해결방법
date:   2023-12-28
last_modified_at: 2023-12-28
category: [Trouble Shooting]
tags: [Springboot IntelliJ]
---



# Error Logs
```
java.lang.IllegalStateException : Failed to introspect Class [org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration] from ClassLoader
...
Caused by : java.lang.NoClassDefFoundError : javax/servlet/DispatcherType
...
Caused by : java.lang.ClassNotFoundException : javax.servlet.DispatcherType
``` 



# Cause by
Maven pom.xml 에서 tomcat의 scope가 provided로 설정되어 있었으나
Eclipse와는 다르게 IntelliJ는 Run Configuration > provided를 무시할지 여부를 별도로 설정해야만 tomcat을 제대로 읽어들인다. 



# Solution
+ scope 제거
<br/>
scope가 필요하지 않은 경우 (내장tomcat으로 돌려도 상관 없는 경우) 제거하고 돌리면 정상구동한다.

*AS-IS*
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>privided</scope>
</dependency>
```

*TO-BE*
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
</dependency>
```

+ Add Dependencies with "provided" scope to classpath 옵션 추가
<br/>
Run/Debug Configuration > Modify options > ‘ADD DEPENDENCIES WITH "PROVIDED" SCOPE TO CLASSPATH’ 옵션 추가 > maven reload > run
<br/>
상기 과정을 통해 구동하면 정상구동한다.


# Refference
[stackoverflow_failed-to-introspect-class-org-springframework-security-config-annotation-web][stackoverflowLink]
> We were seeing this locally because when running Spring Boot application from IntelliJ we didn't have the option 'Include dependencies with "Provided" scope' in the Run/Debug Configuration ticked.

[stackoverflowLink]: https://stackoverflow.com/questions/53714699/failed-to-introspect-class-org-springframework-security-config-annotation-web-c
