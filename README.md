# How to Install and Configure Prometheus and Grafana on Ubuntu?

## How to Install and Configure Prometheus, Grafana, and Node Exporter
- In these instructions, the system hosting the Prometheus server is referred to as the “monitoring server”. The system being monitored is a “client”. It is possible to develop very complicated custom exporters and dashboards using Prometheus and Grafana. However, this guide describes a more straightforward solution for monitoring the most critical client details, including CPU, memory, and I/O usage. It does not require any knowledge of PromQL or any low-level details for either Prometheus or Grafana.

- To configure the end-to-end solution, the following steps are required.

1) Download and install Prometheus on the monitoring system.
2) Configure Prometheus to run as a service.
3) Install Node Exporter on all clients.
4) Configure Prometheus to monitor the clients.
5) Install and deploy the Grafana server.
6) Integrate Grafana and Prometheus.
7) Import a Dashboard for the Node Exporter Statistics.

### This guide is designed for Ubuntu LTS users.


## How to Download and Install Prometheus
Prometheus can be downloaded as a precompiled binary from the GitHub repository. The Prometheus Downloads repository lists the most recent release of Prometheus. The Prometheus GitHub page also provides instructions on how to build Prometheus from the source code or run it as a Docker container. To download Prometheus, follow these steps.

1) Visit the Prometheus downloads and make a note of the most recent release. The most recent LTS release is clearly indicated on the site.

2) Use wget to download Prometheus to the monitoring server. The target link has the format ` https://github.com/prometheus/prometheus/releases/download/v[release]/prometheus-[release].linux-amd64.tar.gz.`

Replace the string [release] with the actual release to download. For example, the following command downloads release 2.37.6.

`wget https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz`

3) Extract the archived Prometheus files.

`tar xvfz prometheus-*.tar.gz`

4) (Optional) After the files have been extracted, delete the archive or move it to a different location for storage.

`rm prometheus-*.tar.gz`

5) Create two new directories for Prometheus to use. The /etc/prometheus directory stores the Prometheus configuration files. The /var/lib/prometheus directory holds application data.

`sudo mkdir /etc/prometheus /var/lib/prometheus`

6) Move into the main directory of the extracted prometheus folder. Substitute the name of the actual directory in place of prometheus-2.37.6.linux-amd64.

`cd prometheus-2.37.6.linux-amd64`

7) Move the prometheus and promtool directories to the /usr/local/bin/ directory. This makes Prometheus accessible to all users.

`sudo mv prometheus promtool /usr/local/bin/`

8) Move the prometheus.yml YAML configuration file to the /etc/prometheus directory.

`sudo mv prometheus.yml /etc/prometheus/prometheus.yml`

9) The consoles and console_libraries directories contain the resources necessary to create customized consoles. This feature is more advanced and is not covered in this guide. However, these files should also be moved to the etc/prometheus directory in case they are ever required.

`sudo mv consoles/ console_libraries/ /etc/prometheus/`

10) Verify that Prometheus is successfully installed using the below command:
    `prometheus --version`

## How to Configure Prometheus as a Service
Although Prometheus can be started and stopped from the command line, it is more convenient to run it as a service using the systemctl utility. This allows it to run in the background.

Before Prometheus can monitor any external systems, additional configuration details must be added to the prometheus.yml file. However, Prometheus is already configured to monitor itself, allowing for a quick sanity test. To configure Prometheus, follow the steps below.

1) Create a prometheus user. The following command creates a system user.

`sudo useradd -rs /bin/false prometheus`

2) Assign ownership of the two directories created in the previous section to the new prometheus user.

`sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus`

3) To allow Prometheus to run as a service, create a prometheus.service file using the following command:

`sudo vi /etc/systemd/system/prometheus.service`

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle \
    --log.level=info

[Install]
WantedBy=multi-user.target
```

- The Wants and After options must be set to network-online.target.
- The User and Group fields must both be set to prometheus.
- The ExecStart parameter explains where to find the prometheus executable and defines the default options.
- The config.file option defines the location of the Prometheus configuration file as /etc/prometheus/prometheus.yml.
- storage.tsdb.path tells Prometheus to store application data in the /var/lib/prometheus/ directory.
- web.listen-address is set to 0.0.0.0:9090, allowing Prometheus to listen for connections on all network interfaces.
- The web.enable-lifecycle option allows users to reload the configuration file without restarting Prometheus.

4) Reload the systemctl daemon.

`sudo systemctl daemon-reload`

5) (Optional) Use systemctl enable to configure the prometheus service to automatically start when the system boots. If this command is not added, Prometheus must be launched manually.

`sudo systemctl enable prometheus`

6) Start the prometheus service and review the status command to ensure it is active.

`sudo systemctl start prometheus`
` systemctl status prometheus`

7) Access the Prometheus web interface and dashboard.
