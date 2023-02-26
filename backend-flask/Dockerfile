FROM python:3.10-slim-buster

WORKDIR /backend-flask


COPY requirements.txt requirements.txt 
RUN pip install --no-cache-dir -r requirements.txt



ENV FLASK_ENV=development


COPY . . 

EXPOSE ${PORT}

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]