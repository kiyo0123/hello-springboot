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
You can skip this when using latest appengine sdk (TODO)
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
ab -n 10000 -c 100 fukudak-java.appspot.com/hello
```


# Spring Boot on GKE

## create cluster
```
gcloud container clusters create \
	--scopes "cloud-platform" \
    --zone=asia-northeast1-a \
    cluster-tokyo
```


## jar package (optional)
```
<packaging>jar</packaging> <!-- jar package for Docker -->
```

```
mvn packag
```

## create Dockerfile 

```
touch Dockerfile
```
1. if you want to run jar file 
```
# Start with a base image containing Java runtime
FROM openjdk:8-jdk-alpine

# Add a volume pointing to /tmp
VOLUME /tmp

# Make port 8080 available to the world outside this container
EXPOSE 8080

# The application's jar file
ARG JAR_FILE=target/demo-0.0.1-SNAPSHOT.jar

# Add the application's jar to the container
ADD ${JAR_FILE} app.jar

# Run the jar file 
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

2. if you want to deploy war file to tomcat
```
FROM tomcat:8.5-alpine
COPY target/demo-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/app.war
RUN sh -c 'touch /usr/local/tomcat/webapps/app.war'
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java -Dspring.profiles.active=docker -Djava.security.egd=file:/dev/./urandom -jar /usr/local/tomcat/webapps/app.war" ]
```


## build Docker images 
```
docker build -t gcr.io/<project-id>/hello-spring .
```

push to Cloud Repository
```
gcloud docker -- push gcr.io/<project-id>/hello-spring
```

or alternatively
```
gcloud auth configure-docker
docker push grc.io/<fukudak-id>/hello-spring
```


## test Docker 
```
docker run -p 8080:8080 -t gcr.io/<project-id>/hello-spring  
```

```
curl localhost:8080/hello
```

## Kubernetes config files

```

```

## run k8s object through interactive mode
create deployments and pods
```
kubectl run hello-spring --image gcr.io/fukudak-java/hello-spring --port 8080
```

create svc
```
kubectl expose deployment hello-spring --type "LoadBalancer"
```


