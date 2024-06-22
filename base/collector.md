---
title: Collector Setup
---
# Collector Setup Guide
## Install

```sh
# Change dir as per need

sudo mkdir -p /opt/otel
sudo chown $(whoami):$(whoami) /opt/otel

cd /tmp
curl -L https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.94.0/otelcol-contrib_0.94.0_linux_amd64.tar.gz -o otelcol.tar.gz
mkdir -p otelcol
tar -xvf otelcol.tar.gz -C otelcol
mv otelcol/otelcol-contrib otelcol/otel-collector
sudo mv otelcol/otel-collector /opt/otel
```
## Config File

::: details
Default Ports
- `4318` - `grpc`
- `4317` - `http`
:::
```sh{5,6}
cat <<EOL > /opt/otel/config.yaml
receivers:
  otlp:
    protocols:
      grpc: 
      http:
processors:
  batch:
exporters:
  logging:
service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
EOL
```
## Systemd Service File

```sh
cat <<EOL | sudo tee /etc/systemd/system/otel-collector.service > /dev/null
[Unit]
Description=OpenTelemetry Collector
After=network.target

[Service]
ExecStart=/opt/otel/otel-collector --config /opt/otel/config.yaml
Restart=always
User=$(whoami)

[Install]
WantedBy=multi-user.target
EOL
```
## Enable Collector Service

```sh
sudo systemctl daemon-reload
sudo systemctl enable otel-collector
sudo systemctl start otel-collector
```
## Verify

```sh
sudo journalctl -u otel-collector -f 

```