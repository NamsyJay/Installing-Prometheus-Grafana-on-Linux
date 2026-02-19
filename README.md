<img width="952" height="400" alt="https___dev-to-uploads s3 amazonaws com_uploads_articles_v77yy73dn8y58ix23rz1-Photoroom" src="https://github.com/user-attachments/assets/43057d6a-2e42-4112-b7b9-d3e13298f522" />

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

## How To Configure Prometheus As A Service
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


## How To Install And Configure Node Exporter On The Client
Before a remote system can be monitored, it must have some type of client to collect the statistics. Several third-party clients are available. However, for ease of use, Prometheus recommends the Node Exporter client. After Node Exporter is installed on a client, the client can be added to the list of servers to scrape in prometheus.yml.

To install Node Exporter, follow these steps. Repeat these instructions for every client.
1) Consult the Node Exporter section of the Prometheus downloads page and determine the latest release.

2) Use wget to download this release. The format for the file is `https://github.com/prometheus/node_exporter/releases/download/v[release_num]/node_exporter-[release_num].linux-amd64.tar.gz.` Replace [release_num] with the number corresponding to the actual release. For example, the following example demonstrates how to download Node Exporter release 1.5.0.

`wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz`

3) Extract the application.

`tar xvfz node_exporter-*.tar.gz`

4) Move the executable to usr/local/bin so it is accessible throughout the system.

`sudo mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin`

5) (Optional) Remove any remaining files.

`rm -r node_exporter-1.5.0.linux-amd64*`

6) There are two ways of running Node Exporter. It can be launched from the terminal using the command node_exporter. Or, it can be activated as a system service. Running it from the terminal is less convenient. But this might not be a problem if the tool is only intended for occasional use. To run Node Exporter manually, use the following command. The terminal outputs details regarding the statistics collection process.

`node_exporter`

7) It is more convenient to run Node Exporter as a service. To run Node Exporter this way, first, create a node_exporter user.

`sudo useradd -rs /bin/false node_exporter`

8) Create a service file for systemctl to use. The file must be named node_exporter.service and should have the following format. Most of the fields are similar to those found in prometheus.service, as described in the previous section.

`sudo vi /etc/systemd/system/node_exporter.service`
```
File: /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

9) (Optional) If you intend to monitor the client on an ongoing basis, use the systemctl enable command to automatically launch Node Exporter at boot time. This continually exposes the system metrics on port 9100. If Node Exporter is only intended for occasional use, do not use the command below.

`sudo systemctl enable node_exporter`

10) Reload the systemctl daemon, start Node Exporter, and verify its status. The service should be active.

```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

node_exporter.service - Node Exporter
Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2023-04-11 13:48:06 UTC; 4s ago

11) Use a web browser to visit port 9100 on the client node, for example, http://local_ip_addr:9100. A page entitled Node Exporter is displayed along with a link reading Metrics. Click the Metrics link and confirm the statistics are being collected. For a detailed explanation of the various statistics, see the Node Exporter Documentation.

## How to Configure Prometheus to Monitor Client Nodes
The client nodes are now ready for monitoring. To add clients to prometheus.yml, follow the steps below:

1) On the monitoring server running Prometheus, open prometheus.yml for editing.

`sudo vi /etc/prometheus/prometheus.yml`

2) Locate the section entitled scrape_configs, which contains a list of jobs. It currently lists a single job named prometheus. This job monitors the local Prometheus task on port 9090. Beneath the prometheus job, add a second job having the job_name of remote_collector. Include the following information.

- A scrape_interval of 10s.
- Inside `static_configs` in the targets attribute, add a bracketed list of the IP addresses to monitor. Separate each entry using a comma.
- Append the port number :9100 to each IP address.
- To enable monitoring of the local server, add an entry for localhost:9100 to the list.
The entry should resemble the following example. Replace remote_addr with the actual IP address of the client.

```
File: /etc/prometheus/prometheus.yml
...
- job_name: "remote_collector"
  scrape_interval: 10s
  static_configs:
    - targets: ["remote_addr:9100"]
```

3) To immediately refresh Prometheus, restart the prometheus service.

`sudo systemctl restart prometheus`

4) Using a web browser, revisit the Prometheus web portal at port 9090 on the monitoring server. Select Status and then Targets. A second link for the remote_collector job is displayed, leading to port 9100 on the client. Click the link to review the statistics.


## How to Install and Deploy the Grafana Server
Prometheus is now collecting statistics from the clients listed in the scrape_configs section of its configuration file. However, the information can only be viewed as a raw data dump. The statistics are difficult to read and not too useful.

Grafana provides an interface for viewing the statistics collected by Prometheus. Install Grafana on the same server running Prometheus and add Prometheus as a data source. Then install one or more panels for interpreting the data. To install and configure Grafana, follow these steps.

1) Install some required utilities using apt.

`sudo apt-get install -y apt-transport-https software-properties-common`

2) Import the Grafana GPG key.

`sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key`

3) Add the Grafana “stable releases” repository.

`echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list`

4) Update the packages in the repository, including the new Grafana package.

`sudo apt-get update`

5) Reload the systemctl daemon.

`sudo systemctl daemon-reload`

6) Enable and start the Grafana server. Using systemctl enable configures the server to launch Grafana when the system boots.
```
sudo systemctl enable grafana-server.service
sudo systemctl start grafana-server
```

7) Verify the status of the Grafana server and ensure it is in the active state.

`sudo systemctl status grafana-server`

# Conclusion
Prometheus is a system monitoring application that polls client systems for key metrics. Each client node must use an exporter to collect and expose the requested data. Prometheus is most effective when used together with the Grafana visualization tool. Grafana imports the metrics from Prometheus and presents them using an intuitive dashboard structure.

To integrate the components, download and install Prometheus on a central server and configure Prometheus as a service. Install the Prometheus Node Exporter on each client to collect the data and configure Prometheus to poll the clients. Install Grafana on the same server as Prometheus and configure Prometheus as a data source.
