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

3. Paste and run the following to add Python code to a new file called myapp.py

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

4. Paste and run the following to add the following dependency to a new **requirements.txt** file.

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

7. Back on your computer, paste and run the following to add build steps to a new **Dockerfile**.

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

8. Authenticate to Docker Hub. Enter your Docker username and access token when prompted.

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

12. Return to Docker Hub in your web browser. Navigate to **Repositories**, then click into your $DOCKERUSER/myapp repository. Verify an image with the tag **v1** has been successfully pushed.

## C. Create and configure a custom Helm chart for your application

1. From within the **myapp/ directory**, create a new directory for writing your helm chart.

```
mkdir helm/
```
```
cd helm/
```

2. Run the following to create a new chart directory structure.

```
helm create myapp
```

3. Change into the **helm/myapp/** directory.

```
cd myapp/
```
```
pwd
```
   
4. Inspect the directory contents. Note the presence of **Chart.yaml**, **values.yaml**, and a directory called **templates**.

```
ls -la
```

5. Inspect the content of Chart.yaml.

```
cat Chart.yaml
```

6. Change into the **templates/** directory and inspect its contents.

```
cd templates/
```
```
ls -la
```

7. Delete the **NOTES.txt** file for now. We'll create and populate a new NOTES.txt later

```
rm NOTES.txt
```

8. Navigate up a level so you are back in the helm/myapp chart root.

```
cd ../
```
```
pwd
```

9. Inspect the default **values.yaml** file. We're going to modify this with our own values.

```
cat values.yaml
```

10. Set an environment variable for your Docker username (in case it was prevously reset).

```
read -p "Docker username: " DOCKERUSER
```

Type your Docker username, then press **Return**.

11. Paste and run the following to replace the default values.yaml configuration with one for our own app.

```yaml
cat << EOF > values.yaml
replicaCount: 1

image:
  repository: $DOCKERUSER/myapp
  pullPolicy: IfNotPresent
  tag: "v1"

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext: {}

securityContext: {}

service:
  type: ClusterIP
  port: 5000

ingress:
  enabled: true
  className: ""
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
  hosts:
    - host: ""
      paths:
        - path: /
          pathType: Prefix
  tls: [] 


resources: {}

livenessProbe:
  httpGet:
    path: /
    port: http
readinessProbe:
  httpGet:
    path: /
    port: http

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

volumes: []

volumeMounts: []

nodeSelector: {}

tolerations: []

affinity: {}
EOF
```

12. Run **helm lint** to do a syntax check on your customized chart. The output should read `1 chart(s) linted, 0 chart(s) failed`.

```
helm lint .
```

13. Navigate up a level into the **helm/** directory.

```
cd ../
```
```
pwd
```

## D. Package and install the custom helm chart

1. From inside the **helm/** directory (not helm/myapp/), run the following to package your helm chart.

```
helm package myapp
```

2. Deploy the helm chart to your cluster.

```
helm install myapp ./myapp-0.1.0.tgz
```

3. Check that **myapp** successfully deployed.

```
helm list
```
```
kubectl get deployments
```
```
kubectl get services
```
```
kubectl get pods
```

4. Test connecting to your application's via the ingress rules set in values.yaml. The output be the welcome message defined in your program.

```
curl http://localhost
```

## E. Create and deploy a new version of the application and helm chart.

1. Navigate up to the *parent* **myapp/** directory.

```
cd ../
```

2. Modify **myapp.py** so the welcome message prints something like "Welcome to My App Version 2!"

3. Build a new version of a Docker image, this time with a **v2** tag.

```
docker build -t $DOCKERUSER/myapp:v2 .
```

4. Push the new image and tag to Docker Hub.

```
docker push $DOCKERUSER/myapp:v2
```

5. In a web browser, navigate to your Docker Hub account and verify a new image with the **v2** tag is there.

6. Back in your terminal, navigate to the **helm/myapp/templates** directory.

```
cd helm/myapp/templates
```

7. Paste and run the following to create a **NOTES.txt** file with post-deployment instructions.

```
cat << EOF > NOTES.txt
You can access My App by connecting to localhost at port 80
E.g. curl http://localhost:80
EOF
```

8. Navigate up a level back into the **helm/myapps** directory.

```
cd ../
```
```
pwd
```

9. Open **Chart.yaml** and change the following parameters to increment the versions of your application and chart, respectively.

```yaml
version: 0.2.0
appVersion: "2.0.0"
```

10. Open **values.yaml**. Change **image.tag** to use the **v2** Docker image.

```yaml
image:
  repository: nicklotz/myapp
  pullPolicy: IfNotPresent
  tag: "v2"  # CHANGE THE TAG HERE TO "v2"
```

11. Run **helm lint** to do a syntax check on your customized chart. The output should read `1 chart(s) linted, 0 chart(s) failed`.

```
helm lint .
```

12. Navigate up a level into the **helm/** directory.

```
cd ../
```
```
pwd
```

13. From inside the **helm/** directory (not helm/myapp/), run the following to package your helm chart. A new **myapp-0.2.0.tgz** should be created.

```
helm package myapp
```

14. Upgrade your deployed helm chart to the new version.

```
helm upgrade myapp myapp --version 0.2.0
```

15. Connect to **myapp** to verify the new version is running.

```
curl http://localhost
```

16. Check the App and Chart versions in `helm list`.

```
helm list
```

17. Cleanup.

```
helm uninstall myapp
```




