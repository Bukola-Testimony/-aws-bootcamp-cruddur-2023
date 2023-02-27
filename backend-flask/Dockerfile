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
# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]