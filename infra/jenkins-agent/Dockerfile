FROM python:3.11.4-slim-bookworm AS python-runtime

FROM jenkins/inbound-agent:3396.ve02b_5cde31b_2-4-jdk21

USER root

COPY --from=python-runtime /usr/local/ /usr/local/

WORKDIR /home/jenkins/agent

COPY requirements.txt /tmp/requirements.txt
RUN python --version \
    && python -m pip install --no-cache-dir --requirement /tmp/requirements.txt \
    && rm /tmp/requirements.txt

USER jenkins
