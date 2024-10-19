FROM driveone/onedrive

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update
RUN apt-get install -y locales

RUN echo "ko_KR.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen ko_KR.UTF-8 && \
    dpkg-reconfigure locales && \
    /usr/sbin/update-locale LANG=ko_KR.UTF-8

ENV LC_ALL ko_KR.UTF-8
