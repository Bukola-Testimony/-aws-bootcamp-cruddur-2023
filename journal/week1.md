# Week 1 — App Containerization

we started off the live stream with guest tutors Edith pucilla and james spurin.

I learnt about docker hub, this is  where docker images are pushed to or pulled from and its free to use

I learnt how to create a dockerfile and wrote my first dockerfile for back-end and front-end. see code below;

## Bank-end Dockerfile
```
FROM python:3.10-slim-buster
# Inside container
# make a new folder inside container
WORKDIR /backend-flask
# Outside container -> Insiide container
# this contains the libraries you want to install to run the app
COPY requirements.txt requirements.txt
# Inside container
# Install the python libraries used for the app
RUN pip3 install -r requirements.txt
# outsiide container -> Inside container
# . means everything in the current directory
# first period . represents /backend-flask (inside container)
# second period . /backend-flask (inside container)
COPY . .
# set environment variables (Env vars)
# Insiide container and will remain set when the container is run
ENV FLASK_ENV=development
EXPOSE ${PORT}
# CMD(command)
# 
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

## Front-end Dockerfile

```
FROM node:16.18
ENV PORT=3000
COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
I worked with gitpod and it is gradually becoming efficient for me.

![](/images/week-1/gitpod-workspaces-dockerfile-backend.png)
![](/images/week-1/Build-python-flask-Docker-image.png)


I ran into some error which I finally debugged using the environment variables

> ErrorPage
![Error 404](/images/week-1/flask-404.png)
> Debugged
![Debugged](/images/week-1/flask-port-json.png)



I built my first image by running this command build -t  backend-flask ./backend-flask in the workspace/aws-bootcamp-cruddur-2023 to build the backend flask.  

Instead of writing multiple docker files as seen above, i was introduced to docker-compose. The purpose of docker-compose file is to allow us to run multiple containers at the same time. see code below;

```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js
# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```
I ran docker-compose up to build both the front-end and backend flask and previewd it on port 3000

### Below are the screen shots of both the frontend and the backend applications as I ran the docker commands also exposing the  ports.

> Backend
![Backend](/images/week-1/crud-backend-container-image.png)
> Frontend
![Frontend](/images/week-1/crud-frontend-container-image.png)



I edited the files following the video tutorial by andrew brown to implement and display notifications 

<img width="1435" alt="Screenshot 2023-02-24 at 7 28 51 PM" src="https://user-images.githubusercontent.com/88699664/221262597-b86996df-0ff3-444c-a0c2-986bde5d00b2.png">

I edited the docker-compose file and added the code for dynamodb and posgres and ran docker-compose up. subsequetly ran the following command in my aws cli below;

# Create a table

```
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```

# Create an Item

```
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL  
```

# List Tables

```
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

# Get Records

```
aws dynamodb scan --table-name cruddur_cruds --query "Items" --endpoint-url http://localhost:8000
```

i also installed postgres client into the gitpod.yml file and made sure it installs on start gitpod startup

i checked that the postgress was working by logging into postgres by running `psql -Upostgres --host localhost` and inputed the password i created in the. 
i also installed postgres client into the gitpod.yml file and made sure it installs on start gitpod startupclient extention. inputed /l to check the list of tables

I learnt the top 10 container security best practices 