# This image provides a jupyter notebook setup

FROM registry.access.redhat.com/rhscl/python-36-rhel7

LABEL maintainer="Stefan Klingelschmitt <stefan.klingelschmitt@boehringer-ingelheim.com>"

ARG nexusHostWithBasicAuth
ARG nexusHostWithoutScheme

WORKDIR /app

ENV PYTHONPATH=$PYTHONPATH:/app

COPY requirements.txt /app

USER 1001
# From load pip install for caching docker build layers
RUN if [ ! -z ${nexusHostWithBasicAuth} ]; \
    then pip install -i ${nexusHostWithBasicAuth}/repository/pypi-all/simple --trusted-host ${nexusHostWithoutScheme} --upgrade pip && pip install -i ${nexusHostWithBasicAuth}/repository/pypi-all/simple --trusted-host ${nexusHostWithoutScheme} -r requirements.txt; \
    else pip install --upgrade pip && pip install -r requirements.txt; \
    fi


USER root

COPY run.sh /app/run.sh
COPY jupyter_notebook_config.json /opt/app-root/src/.jupyter/jupyter_notebook_config.json

RUN chgrp -R 0 /app && \
    chmod -R g=u /app && \
    chmod +x /app/run.sh && \
    chmod g+w /etc/passwd

USER 1001


EXPOSE 8080

ENTRYPOINT [ "/app/run.sh" ]

