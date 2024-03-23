# Lab 05: Templates and Built-in Functions

## A. Create and Dockerize a simple Spring app

1. Install Java (OpenJDK) if you don't have it. This lab assumes Java 17.

```
sudo apt install openjdk-17-jre-headless
```

2. In a web browser, go to [https://start.spring.io](https://start.spring.io) to bootstrap a new Spring App. Select the following options:
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
package com.example.myspringapp;

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

10. Authenticate to Docker Hub. Enter your Docker username and access token when prompted.

```
docker login
```

11. Set an environment variable for your Docker username.

```
read -p "Docker username: " DOCKERUSER
```

Type your Docker username, then press **Return**.


12. Build your docker image.

```
docker build -t $DOCKERUSER/myspringapp:0.0.1 .
```

13. **OPTIONAL**: Test the container locally. Be sure to clean up when done.

```
docker run -d -p 8080:8080 myspringapp
```
```
curl https://localhost:8080
```
```
docker stop <CONTAINER_ID>
```
```
docker rm <CONTAINER_ID>
```

14. Push the image to Docker Hub.

```
docker push $DOCKERUSER/myspringapp:0.0.1
```

15. Return to Docker Hub in your web browser. Navigate to **Repositories**, then click into your $DOCKERUSER/myspringapp repository. Verify an image with the tag **0.0.1** has been successfully pushed.

## B. Customize and deploy a Helm chart for the app

1. From the **myspringapp/** directory, initialize a new Helm chart.

```
mkdir helm/
cd helm/
```
```
helm create myspringapp
cd myspringapp/
```

2. Past and run the following to create a customized **values.yaml**.

```yaml
cat << EOF > values.yaml
replicaCount: 1

image:
  repository: spring-boot-app
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 8080

appConfig:
  environment: "development"
EOF
```
```
cat values.yaml
```

3. Paste and run the following to create a customized **deployment.yaml**. Note the use of piping and built-in template functions.

```yaml
cat << EOF > templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "spring-boot-app.fullname" . | trunc 63 | trimSuffix "-" }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "spring-boot-app.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "spring-boot-app.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: {{ .Values.appConfig.environment | quote }}
EOF
```
```
cat -n templates/deployment.yaml
```

4. Deploy the Helm chart.


