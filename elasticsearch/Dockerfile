# escape=`
FROM microsoft/windowsservercore:ltsc2016 AS downloader
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ARG ES_VERSION="7.10.2"
ENV ES_HOME="C:\elasticsearch" `
    ES_ROOT_URL="https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss"

# First two commands enable TLS1.2
RUN Set-ItemProperty -Path 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord; `
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord; `
    Invoke-WebRequest "$env:ES_ROOT_URL-$env:ES_VERSION-windows-x86_64.zip.sha512" -OutFile 'elasticsearch.zip.sha' -UseBasicParsing; `
    $env:ES_SHA = (Get-Content -Raw elasticsearch.zip.sha).Split(' ')[0]; `
    Invoke-WebRequest "$env:ES_ROOT_URL-$env:ES_VERSION-windows-x86_64.zip" -OutFile 'elasticsearch.zip' -UseBasicParsing; `
    if ((Get-FileHash elasticsearch.zip -Algorithm sha512).Hash.ToLower() -ne $env:ES_SHA) {exit 1}; `
    Expand-Archive elasticsearch.zip -DestinationPath C:\ ; `
    Move-Item c:/elasticsearch-$($env:ES_VERSION) $env:ES_HOME;

# Elasticsearch
FROM openjdk:11-nanoserver-sac2016
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop';"]

ARG ES_VERSION="7.10.2"
ENV ES_VERSION=${ES_VERSION} `
    ES_HOME="C:\elasticsearch" `
    ES_JAVA_OPTS="-Xms800m -Xmx800m"

# Volume and drive mount
RUN md C:\elasticsearch\data
# RUN Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices' -Name 'G:' -Value '\??\C:\data' -Type String

EXPOSE 9200 9300
SHELL ["cmd", "/S", "/C"]
CMD ".\\bin\\elasticsearch.bat"
# CMD ping localhost -t

WORKDIR $ES_HOME
COPY --from=downloader C:\elasticsearch\ .
COPY config ./config

#RUN del c:\elasticsearch\logs /F /S /Q

HEALTHCHECK --interval=5s `
 CMD powershell -command `
    try { `
     $content = (iwr -useb http://localhost:9200/_cat/health).Content; `
     $health = $content.Split(' ')[3]; `
     if ($health -eq 'green' -or $health -eq 'yellow') { return 0 } `
     else { return 1 }; `
    } catch { return 1 }