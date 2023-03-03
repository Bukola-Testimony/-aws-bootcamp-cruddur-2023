# Week 1 — App Containerization

- we started off the live stream with guest tutors Edith pucilla and james spurin.

- I learnt about docker hub, this is  where docker images are pushed to or pulled from and its free to use

- I learnt how to create a dockerfile and wrote my first dockerfile for back-end and front-end. see code below;

## Backend Dockerfile
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
- I worked with gitpod which has become pretty efficient for me.

![](/images/week-1/gitpod-workspaces-dockerfile-backend.png)
![](/images/week-1/Build-python-flask-Docker-image.png)

<br>

- I ran into some errors which I finally debugged using the environment variables

> ErrorPage
![Error 404](/images/week-1/flask-404.png)
> Debugged
![Debugged](/images/week-1/flask-port-json.png)



- I built my first image by running the below command to build the backend flask: 
```
build -t  backend-flask ./backend-flask 
```
  

- I was also introduced to docker-compose. The purpose of docker-compose file is to allow us to run multiple containers at the same time. see code below;

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
<br>

- I ran "docker-compose up" to build both the front-end and backend flask and previewd it on port 3000

- Below are the screen shots of both the frontend and the backend applications as I ran the docker commands also exposing the  ports.

<br>

## Backend
![Backend](/images/week-1/crud-backend-container-image.png)
## Frontend
![Frontend](/images/week-1/crud-frontend-container-image.png)

<br>

- I edited the files following the video tutorial by andrew brown to implement and display the notifications page. 
- See screenshot of my modified Notifications page. 

<br>

## Notifications Page
![Notifications-page](/images/week-1/edited-notifications-page.png)

<br>

- I edited the docker-compose file and added the code for dynamodb and posgres and ran "docker-compose up". 
- Subsequetly I ran the following command in my aws cli:

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

![dynamodb list-tables](/images/week-1/dynamodb-list-tables.png)


# Get Records

```
aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
```

![dynamodb table records](/images/week-1/dynamodb-table-records.png)

- I installed postgress using the following code. I also added the code to gitpod.yml

```yaml
- name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```


- I connected to my postgress client with the following command:
- I checked the tables too. All was working well.

```
psql -Upostgres  -h localhost 

```
Password: password

![Postgress-table](/images/week-1/postgress-table.png)

I learnt the top 10 container security best practices 