# Use latest debian unstable
FROM debian:sid-slim
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get -y dist-upgrade

# Install few extra packages needed to clone repository and run web HTTP services
RUN apt-get install -y git \
                       apache2 \
                       nano \
                       libapache2-mod-wsgi-py3 \
                       build-essential \
                       -o APT::Install-Suggests=0 \
                       -o APT::Install-Recommends=0
# build-eesential is not needed,
# but adding it anyway for many reason.
# is is useful when playing with the image
# and debug packages that requires to be built from src

# Install minimal debian python build environment
RUN apt-get install -y dpkg-dev  \
                       fakeroot  \
                       lintian  \
                       python3-setuptools  \
                       python3-wheel  \
                       python3-pip  \
                       libpq-dev \
                       postgresql-server-dev-all \
                       python3-dev libpython3-dev \
                       -o APT::Install-Suggests=0 \
                       -o APT::Install-Recommends=0

RUN useradd -ms /bin/bash pycsw

RUN cd /var/www/ && git clone git://github.com/geopython/pycsw.git \
  && pip3 install --upgrade pip setuptools \
  && pip3 install --requirement pycsw/requirements.txt \
  && pip3 install --requirement pycsw/requirements-standalone.txt \
  && pip3 install --requirement pycsw/requirements-pg.txt \
  && pip3 install gunicorn \
  && cd pycsw && python3 setup.py build && python3 setup.py install

ENV PYCSW_CONFIG=/etc/pycsw/pycsw.cfg

RUN mkdir /etc/pycsw \
  && mv /var/www/pycsw/docker/pycsw.cfg ${PYCSW_CONFIG} \
  && mkdir /var/lib/pycsw \
  && chown pycsw:pycsw /var/lib/pycsw \
  && cp /var/www/pycsw/docker/entrypoint.py /usr/local/bin/entrypoint.py \
  && chmod a+x /usr/local/bin/entrypoint.py

# Add xml test data
# RUN mkdir -p /var/www/pycsw/xml/data
# ADD docker_test/test.xml /var/www/pycsw/xml/data/test.xml

# COPY sample configuration to run pydap as wsgi application under apache
ADD docker_test/pycsw.conf /etc/apache2/sites-enabled/pycsw.conf

#RUN update-rc.d apache2 defaults
#EXPOSE 80

WORKDIR /home/pycsw

USER pycsw


EXPOSE 8000

ENTRYPOINT [\
  "python3", \
  "/usr/local/bin/entrypoint.py" \
]