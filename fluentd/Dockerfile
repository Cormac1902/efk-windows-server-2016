# fluentd/Dockerfile

FROM fluent/fluentd:v1.11-windows-2016
USER root
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "3.5.2"]
USER fluent