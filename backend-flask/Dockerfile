FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY script.py .
COPY requirements.txt requirements.txt 


RUN pip install -r requirements.txt
RUN chmod +x run_flask.sh


ENV FLASK_ENV=development

COPY . .

EXPOSE ${PORT}

CMD ["/usr/local/bin/run_flask.sh"]
