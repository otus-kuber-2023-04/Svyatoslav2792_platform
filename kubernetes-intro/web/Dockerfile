FROM alpine:3.5
RUN apk update
RUN apk add py2-pip
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/
EXPOSE 8000
USER 1001
CMD ["python", "/usr/src/app/app.py"]