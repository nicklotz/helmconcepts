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

4. Open myapp.py and paste the following Python code.

```python
# myapp.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def welcome():
    return 'Welcome to My Web App!'

if __name__ == '__main__':
    app.run(host='0.0.0.0')

```

5. Save and close myapp.py.

6. Create a new file called **requirements.txt**.

```
touch requirements.txt
```

7. Open requirements.txt and add the following.

```
Flask==3.0.2
```

8. Save and close requirements.txt

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

8. Open Dockerfile and paste the following content.

```
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "./myapp.py"]
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

12. Create/initialize a helm chart for myapp.

```
helm create myapp-chart
```


