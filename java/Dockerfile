FROM debian
MAINTAINER He Bai <bai.he@outlook.com>
RUN apt-get update && apt-get install -y curl
RUN curl -SL https://www.reucon.net/cdn/java/jdk-7u76-linux-x64.tar.gz | tar -xzC /opt/
WORKDIR /opt
ENV JAVA_HOME /opt/jdk1.7.0_76
ENV PATH ${JAVA_HOME}/bin:${PATH}
RUN echo ${PATH}
