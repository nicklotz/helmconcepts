# Lab 02: Create a Custom Helm Chart

## A. Create a basic web app

1. Create an application directory.

```
mkdir myapp/
```

2. Change into the application directory.

```
cd myapp/
```

3. Create a new file called **myapp.py**

```
touch myapp.py
```

4. Paste and run the following to add Python code to myapp.py

```python
cat << EOF > myapp.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def welcome():
    return 'Welcome to My Web App!'

if __name__ == '__main__':
    app.run(host='0.0.0.0')
EOF
```

5. Create a new file called **requirements.txt**.

```
touch requirements.txt
```

7. Add the following to requirements.txt.

```
echo "Flask==3.0.2" > requirements.txt
```

## B. Dockerize your application

1. Create a free account on [Docker Hub](https://hub.docker.com) if you don't already have one.

2. Once logged into Docker Hub, navigate to your profile icon, then click **My Account**.

3. Select **Security** from the left sidebar.

4. Under **Access Tokens**, click **New Access Token**.

5. Give the access token a name and make sure the permissions are set to **Read, Write, Delete**. Click **Generate**.

6. Copy the token's value and paste in a safe place.

7. Back on your computer, inside the myapp/ directory, create a Dockerfile.

```
touch Dockerfile
```

8. Paste the following followed by **Enter** to modify the Dockerfile. 

```
cat << EOF > Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "./myapp.py"]
EOF
```

9. Log into Docker Hub. Enter your Docker username and access token when prompted.

```
docker login
```

9. Set an environment variable for your Docker username.

```
read -p "Docker username: " DOCKERUSER
```

Type your Docker username, then press **Return**.


10. Build your docker image.

```
docker build -t $DOCKERUSER/myapp:v1 .
```

11. Push the image to Docker Hub.

```
docker push $DOCKERUSER/myapp:v1
```

## C. Create and configure a custom Helm chart.

1. Create/initialize a helm chart for myapp.

```
helm create myapp-chart
```

2. Navigate into the myapp-chart directory.

```
cd myapp-chart
```

3. Paste and execute the following in your terminal to update **Chart.yaml** with chart metadata.

```yaml
cat << EOF >> Chart.yaml
apiVersion: v2
name: myapp
description: A Helm chart for Kubernetes to deploy My Python Application
type: application
version: 0.1.0
appVersion: "1.0"
EOF
```

4. Set configurable chart parameters in **values.yaml**

```yaml
cat << EOF > values.yaml 
replicaCount: 1

image:
  repository: $DOCKERUSER/myapp
  pullPolicy: IfNotPresent
  tag: "v1"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

resources: {}
EOF
```

5. Edit **deployment.yaml** to use values from **values.yaml**.

```
cat << EOF > templates/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "myapp.name" . }}
    spec:
      containers:
        - name: myapp
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 80
EOF
```

6. Edit **service.yaml** to use parameters from **values.yaml**.

```yaml
cat << EOF > templates/service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app.kubernetes.io/name: {{ include "myapp.name" . }}
EOF
```

7. Specify the chart's **ingress.yaml** to use the values from **values.yaml**.

```yaml
cat << EOF > templates/ingress.yaml 
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "myapp.fullname" . }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
{{- end }}
EOF
```

7. 

