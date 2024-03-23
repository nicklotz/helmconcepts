# Lab 05: Templates and Built-in Functions

## A. Create and Dockerize a simple Spring app

1. Install Java (OpenJDK) if you don't have it. This lab assumes Java 17.

```
sudo apt install openjdk-17-jre-headless
```

2. In a web browser, go to [https://start.spring.io] to bootstrap a new Spring App. Select the following options:
    - Project: **Maven**
    - Language: **Java**
    - Spring Boot: **3.2.4**
    - Project Metadata:
        - Group: **com.example**
        - Artifact: **myspringapp**
        - Description: **Simple spring boot app**
        - Packaging: **jar**
        - Java: 17

3. On the right side of the page, next to **Dependencies**, click **Add Dependencies**.

4. Search for and select **Spring Web**.

5. From the bottom of the page, click **Generate**. That should download a file called **myspringapp.zip** to your local computer.

6. Extract **myspringapp.zip** and navigate into the project root directory.

```
unzip myspringapp.zip
```
```
rm myspringapp.zip
```
```
cd myspringapp/
```

7. Paste and run the following to create a new Java class in the project called **HelloController** that serves a welcome message.

```java
cat << EOF > src/main/java/com/example/myspringapp/HelloController.java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String index() {
        return "Hello, Spring Boot!";
    }
}
EOF
```
```
cat -n src/main/java/com/example/myspringapp/HelloController.java
```

8. Build the app with the provided Maven wrapper.

```
./mvnw package
```

9. Paste and run the following to create a new Dockerfile that containerizes the application.

```Dockerfile
cat << EOF > Dockerfile
FROM openjdk:17.0.2-slim-buster
COPY target/myspringapp-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
EOF
```

10. 

