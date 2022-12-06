---
permalink: /dockerizing-flask-app
---

# **Dockerize Flask App**

# Prerequisites

To complete this tutorial, you will need the following:

- Docker installed on your computer
- A text editor for writing code
- A basic knowledge of Python

## Create a Basic Flask App

Create a file called `app.py` in your project directory. This will contain the basic code for your flask app.

```py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

## Create a Dockerfile

Create a file called `Dockerfile` in your project directory. This will contain the instructions for creating the Docker image.

```dockerfile
FROM python:3.7.7-slim
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

## Create a Requirements File

Create a file called `requirements.txt` in your project directory. This will list all of the packages that your application needs in order to run.

```sh
$ pip freeze > requirements.txt
```

## Build the Docker Image

Run the following command from the project directory to build your Docker image.

`docker build -t my-flask-app .`

## Run the Docker Image

Run the following command to start the Docker container with your new image.

`docker run -d -p 5000:5000 my-flask-app`

Your application should now be running on port 5000 of your local machine.
