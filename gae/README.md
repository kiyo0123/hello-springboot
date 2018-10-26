# Spring Boot on GAE


## Configure Spring Boot Project
```
curl https://start.spring.io/starter.tgz -d dependencies=web \
-d javaVersion=1.8 \
-d packaging=war \
-d applicationName=HelloAppEngine \
-d language=java \
-d type=maven-project \
 | tar -xzvf -
```

## Modify pom.xml

Remove spring-boot-starter-tomcat and add javax.servlet-api. This is because GAE is using Jetty while default Spring Boot uses Tomcat.  
```
		<!-- remove below 
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		--> 
		<!-- add below -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
		</dependency>
```

Add maven GAE plug-in. 

```
			 <plugin>
				<groupId>com.google.cloud.tools</groupId>
				<artifactId>appengine-maven-plugin</artifactId>
				<version>1.3.2</version>
				<configuration>
					<deploy.promote>true</deploy.promote>
					<deploy.stopPreviousVersion>true</deploy.stopPreviousVersion>
				</configuration>
			</plugin>

```

## Add GAE configulation file
Add appengine_web.xml for deployment descriptor.
```
$ mkdir -p src/main/webapp/WEB-INF/
$ touch src/main/webapp/WEB-INF/appengine-web.xml
```

Add following context to appengine-web.xml.
```
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
  <version>1</version>
  <threadsafe>true</threadsafe>
  <runtime>java8</runtime>
</appengine-web-app>
```

## Write Application

Replace HelloAppEngineApplication.java with following content.
```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class HelloAppEngine {

	public static void main(String[] args) {
		SpringApplication.run(HelloAppEngine.class, args);
	}

	@GetMapping("/hello")
	public String hello() {
		return "Hello Spring Boot on GAE!! JSUG";
	}
}

```

## Test Application 

```
$ mvn clean appengine:run
```

access to localhost:8080/hello

## Deploy to GAE
```
$ mvn appengine:deploy
```

## Test GAE
access to https://<project-id>.appspot.com/hello

## load test
```
ab -n 10000 -c 100 fukudak-java.appspot.com/he
```


