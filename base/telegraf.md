---
title: Telegraf Setup
---
# Telegraf Setup Guide
## Install

```bash
# Change dir as per need

sudo mkdir -p /opt/telegraf
sudo chown $(whoami):$(whoami) /opt/telegraf
cd /tmp
curl -L https://dl.influxdata.com/telegraf/releases/telegraf-1.30.0_linux_amd64.tar.gz -o telegraf.tar.gz
mkdir -p telegraf
tar -xvf telegraf.tar.gz -C telegraf
sudo mv telegraf/telegraf-1.30.0/usr/bin/telegraf /opt/telegraf

```
## Config File


```bash
cat <<EOL > /opt/telegraf/telegraf.conf
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 100
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_jitter = "0s"
  precision = ""
  debug = false
  quiet = false
  logfile = ""
  hostname = ""
  omit_hostname = false

[[inputs.cpu]]
[[outputs.file]]
  files = ["stdout"]
EOL
```
## Systemd Service File

```bash
cat <<EOL | sudo tee /etc/systemd/system/telegraf.service > /dev/null
[Unit]
Description=Telegraf Agent
After=network.target

[Service]
ExecStart=/opt/telegraf/telegraf --config /opt/telegraf/telegraf.conf
Restart=always
User=$(whoami)

[Install]
WantedBy=multi-user.target
EOL
```
## Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable telegraf
sudo systemctl start telegraf
```
## Verify

```bash
sudo journalctl -u telegraf -f -o cat
```
```bash
cpu,cpu=cpu-total,host=<host-name> usage_system=0.7156758114990167,usage_nice=0,usage_iowait=0.07533429600880925,usage_irq=0,usage_steal=0,usage_guest=0,usage_guest_nice=0,usage_user=1.946135978473559,usage_idle=97.04312888384675,usage_softirq=0.21972502980644198 1718986470000000000
....
```