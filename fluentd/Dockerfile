# fluentd/Dockerfile

FROM fluent/fluentd:v1.11-windows-2016

RUN powershell -Command "gem install fluent-plugin-elasticsearch --no-document --version 4.3.3"
RUN powershell -Command "Remove-Item -Path C:\fluent\conf -Force -Recurse"