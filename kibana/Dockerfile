# escape=`
FROM microsoft/windowsservercore:ltsc2016 AS downloader
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ARG KIBANA_VERSION="7.10.2"
ENV KIBANA_HOME="C:\kibana" `
	KIBANA_ROOT_URL="https://artifacts.elastic.co/downloads/kibana/kibana-oss"

# First two commands enable TLS1.2
RUN Set-ItemProperty -Path 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord; `
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord; `
    Invoke-WebRequest "$env:KIBANA_ROOT_URL-$env:KIBANA_VERSION-windows-x86_64.zip.sha512" -OutFile 'kibana.zip.sha' -UseBasicParsing; `
    $env:KIBANA_SHA = (Get-Content -Raw kibana.zip.sha).Split(' ')[0]; `
    Invoke-WebRequest "$env:KIBANA_ROOT_URL-$env:KIBANA_VERSION-windows-x86_64.zip" -OutFile 'kibana.zip' -UseBasicParsing; `
    if ((Get-FileHash kibana.zip -Algorithm sha512).Hash.ToLower() -ne $env:KIBANA_SHA) {exit 1}; `
    Expand-Archive kibana.zip -DestinationPath C:\; `
    Move-Item c:/kibana-$($env:KIBANA_VERSION)-windows-x86_64 $env:KIBANA_HOME;

# Kibana
FROM microsoft/windowsservercore:ltsc2016
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop';"]

ARG KIBANA_VERSION="7.10.2"
ENV KIBANA_VERSION=${KIBANA_VERSION} `
    KIBANA_HOME="C:\kibana"

EXPOSE 5601
ENTRYPOINT ["powershell"]
CMD ["./init.ps1"]

WORKDIR C:/kibana
COPY init.ps1 .
COPY --from=downloader C:\kibana\ .

# Default configuration for host & Elasticsearch URL
RUN	(Get-Content ./config/kibana.yml) -replace '#server.host: \"localhost\"', 'server.host: \"0.0.0.0\"' | Set-Content ./config/kibana.yml; `
	(Get-Content ./config/kibana.yml) -replace '#elasticsearch.hosts: \[\"http://localhost:9200\"]', 'elasticsearch.hosts: ["http://elasticsearch:9200"]' | Set-Content ./config/kibana.yml;

HEALTHCHECK --start-period=30s --interval=10s --retries=5 `
 CMD powershell -command `
    try { `
     $response = iwr -useb http://localhost:5601/app/kibana; `
     if ($response.StatusCode -eq 200) { return 0} `
     else {return 1}; `
    } catch { return 1 }