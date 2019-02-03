FROM bde2020/spark-submit:2.4.0-hadoop2.7

MAINTAINER Cecile Tonglet <cecile.tonglet@tenforce.com>

COPY template.sh /

RUN apt-get update && apt-get install -y python3-dev gcc
RUN pip3 install --upgrade setuptools pip
RUN mkdir /app

# Copy the requirements.txt first, for separate dependency resolving and downloading
ONBUILD COPY requirements.txt /app/
ONBUILD RUN cd /app \
      && pip3 install -r requirements.txt

# Copy the source code
ONBUILD COPY . /app

CMD ["/bin/bash", "/template.sh"]
