FROM python:3.7-alpine

RUN apk update && \
    apk add postgresql-libs postgresql-dev graphviz-dev gcc musl-dev

WORKDIR /app 

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]