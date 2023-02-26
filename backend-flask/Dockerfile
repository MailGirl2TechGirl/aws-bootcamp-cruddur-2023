FROM python:3.10-slim-buster

WORKDIR /backend-flask


COPY requirements.txt requirements.txt 
RUN pip install -r requirements.txt



COPY . run_flask.sh 
RUN chmod +x run_flask.sh


ENV FLASK_ENV=development


COPY . .

EXPOSE ${PORT}

CMD ["/bin/bash", "run_flask.sh"]