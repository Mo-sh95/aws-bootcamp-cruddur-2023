FROM python:3.10-slim-buster
WORKDIR /backend-flask
COPY requirements.txt requirements.txt
RUN apt-get update && pip3 install -r requirements.txt && apt-get install -y curl gcc
COPY . .
ENV FLASK_ENV=development
EXPOSE ${PORT}
CMD ["sh","-c","./script.sh"]
