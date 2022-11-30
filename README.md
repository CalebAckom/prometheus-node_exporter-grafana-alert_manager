# Prometheus NodeExporter Grafana Alertmanager Setup
This is a documentation on setting up Prometheus Node Exporter Grafana and Alertmanager on Ubuntu 20.04

Table of Contents:

- [Prometheus](#installing-prometheus)
- [Node-Exporter](#installing-node-exporter)
- [Grafana](#installing-grafana)
- [Alertmanager](#installing-alertmanager)

**For the following reasons, dedicated Linux users or system accounts should be created for each service**

1. To reduce the impact in a case of an incident with a service
2. To simplify administration because it is easier to track down what resources belong to which service.

>## Installing Prometheus
1. Create a user for Prometheus service

```
sudo useradd --system --no-create--home --shell /bin/false prometheus
```

**useradd:** A unix command to create a new user or update default new user information

**--system:** A flag to create a system account

**--no-create-home:** A flag to avoid creating a home directory along with the account creation

**--shell /bin/false:** To prevent user logging in

**prometheus:** User name

2. Download Prometheus with ***wget*** command
NB: Check the latest version of [Prometheus](https://prometheus.io/download/)

At the time of this documentation, the latest version is 2.40.2
```
wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.40.2.linux-amd64.tar.gz
```

3. Extract the files from the archive file
```
tar -xvf prometheus-2.40.2.linux-amd64.tar.gz
```

4. Create a ***/data***  directory to serve as a mounted disk
```
sudo mkdir -p /data /etc/prometheus
```

5. Change directory to the extracted Prometheus directory
```
cd prometheus-2.40.2.linux-amd64
```

6. First, move the ```prometheus``` binary and ```promtool``` to the ```/usr/local/bin/``` directory.

``` Promtool``` is used to check for configuration files and Prometheus rules
```
sudo mv prometheus promtool /usr/local/bin/
```

7. Next, move the ```console libraries``` to the Prometheus configuration directory.

Console templates allow for the creation of arbitary consoles using the Go templating language. This is an optional step.
```
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

8. Finally, move the main sample (example) Prometheus configuration file.
```
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

9. Set ownership for ```/etc/prometheus/``` and ```data``` directories, to avoid permission issues
```
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

10. You can choose to delete the Prometheus archive and directory
```
cd
rm -rf prometheus*
```

To verify the Prometheus version
```
prometheus --version
```
To get more information
```
prometheus --help
```

11. Create a systemd unit configuration file. ***systemd*** is a system and service manager for Linux OSs
```
sudo nano /etc/systemd/system/prometheus.service
```
Paste the content in [prometheus.service](./configFiles/prometheus.service "target=_blank")

**RestartSec**: Configures the time to sleep before restarting the service

**User** and **Group**: Linux user and group to start Proometheus service

**--config.file=/etc/prometheus/prometheus.yml**: Path to main Prometheus configuration file

**--storage-tsdb.path=/data**: Prometheus data storage location

**--web.listen-address=0.0.0.0:9090**: Configures to listen on all network interfaces

**--web.enable-lifecycle**: Allows to manage Prometheus. An example is to reload without restarting the service

## Other Operations
Enable the Prometheus
```
sudo systemctl enable prometheus
```
Start Prometheus
```
sudo systemctl start prometheus
```
Check Status of Prometheus service
```
sudo systemctl status prometheus
```
Search for errors with ```journalctl``` command
```
journalctl -u prometheus -f --no-pager
```

**You can now access the Service via Browser**
```
http://<ip-address>:9090
```

>## Installing Node Exporter
Node Exporter is used to collect Linux system metrics like CPU load and disk I/O

1. Just like Prometheus, create a system user for Node Exporter

```
sudo useradd --system --no-create-home --shell /bin/false node_exporter
```

2. Download Node Exporter 

NB: Check the latest version of [Node-Exporter](https://prometheus.io/download/)

At the time of this documentation, the latest version is 1.4.0
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.4.0.linux-amd64.tar.gz
```

3. Extract the files from the archive file
```
tar -xvf node_exporter-1.4.0.tar.gz
```

4. Move binary to the /usr/local/bin
```
sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
```

5. Delete node_exporter archive and folder (***optional***)
```
rm -rf node_exporter*
```

Verify the version
```
node_exporter --version
```
More information
```
node_exporter --help
```

6. Create a systemd unit configuration file
```
sudo nano /etc/systemd/system/node_exporter.service
```
Paste the content in [node_exporter.service](./configFiles/node_exporter.service "target=_blank")

7. Create a static target for Node Exporter service in Prometheus configuration
```
sudo nano /etc/prometheus/prometheus.yml
```

```
...
    - job_name: node_exporter
      static_configs:
        - targets: ['localhost:9100']
```

NB: Node Exporter, by default, runs on port 9100

8. Check to verify the configuration is valid
```
promtool check config /etc/prometheus/prometheus.yml
```
9. Use POST request to reload the config
```
curl -X POST http://localhost:9090/-/reload
```

## Other Operations
Enable the Node Exporter
```
sudo systemctl enable node_exporter
```
Start Node Exporter
```
sudo systemctl start node_exporter
```
Check Status of Node Exporter service
```
sudo systemctl status node_exporter
```
Search for errors with ```journalctl``` command
```
journalctl -u node_exporter -f --no-pager
```

>## Installing Grafana
We will use Grafana to visualize the metrics.

1. Install dependencies
```
sudo apt-get install -y apt-transport-https software-properties-common
```

2. Add GPG Key
GPG, short for GNU Privacy Guard, key is a public key that allows for the secure
transmission of information between parties and can be used to verify that the origin
of a message is genuine. - Digital Ocean
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

3. Add this repository for stable releases
```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

4. Now, update and install Grafana
```
sudo apt-get update
```
```
sudo apt-get install -y grafana
```

5. Enable the Grafana service
```
sudo systemctl enable grafana-server
```

6. Start Grafana
```
sudo systemctl start grafana-server
```

## Other Operations
Check the status
```
sudo systemctl status grafana-server
```

Go to ```http://<ip-address>:3000``` and log in

Default credentials, Username is **admin** and Password is **admin** as well

You can now choose your password after a successful login

7. To visualize the metrics, a data source must be first added
There are two (2) ways

## With Grafana's UI
Click ```Add data source```

Select ```Prometheus```

For URL, input ```http://localhost:9090```

Click ```Save and test```. You should get a ```Data source is working``` message.

## As Code
1. Create a new ```datasources.yml``` file
```
sudo nano /etc/grafana/provisioning/datasources/datasources.yaml
```

2. Paste the content in 
Paste the content in [datasources.yml](./configFiles/datasources.yml "target=_blank")

3. Restart Grafana server to reload the configuration
```
sudo systemctl restart grafana-server
```

>## Installing Alertmanager
Alertmanager, as the name depicts, is used to send alerts. These alerts can be to receiver integrations
such as emails, Slack channels, PagerDuty, etc.

For high availability, you can set up multiple Alertmanagers.

1. Create a user for Alertmanager service
```
sudo useradd --system --no-create-home --shell /bin/false alertmanager
```

2. Download Alertmanager
NB: Check the latest version of [Alertmanager](https://prometheus.io/download/)

At the time of this documentation, the latest version is 2.40.0
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
```

3. Extract Alertmanager binary
```
tar -xvf alertmanager-0.24.0.linux-amd64.tar.gz
```

4. Create storage

We need storage for Alertmanager. The default is ```data/```.
This is used to store Alertmanager's notification states and silences
Without this state (or if you wipe it), Alertmanager would not know across restarts what silences were created or what notifications were already sent

```
sudo mkdir -p /alertmanager-data /etc/alertmanager
```

5. Move Alertmanager's binary to the local bin directory
```
sudo mv alertmanager-0.23.0.linux-amd64/alertmanager /usr/local/bin/
```

6. Copy sample config file
```
sudo mv alertmanager-0.23.0.linux-amd64/alertmanager.yml /etc/alertmanager/
```

7. Remove archive and directory
```
rm -rf alertmanager*
```

8. Create a systemd unit configuration file
```
sudo nano /etc/systemd/system/alertmanager.service
```
Paste the content in [alertmanager.service](./configFiles/alertmanager.service "target=_blank")

9. Enable Alertmanager
```
sudo systemctl enable alertmanager
```

10. Start Alertmanager
```
sudo systemctl start alertmanager
```

>NB: Alertmanager is exposed on port 9093

11. Create alert rule

For this documentation, we will create a simple alert

An alert which will be triggered when a job or service connected to Prometheus is down for more than a minute
```
sudo nano /etc/prometheus/alert_rule.yml
```
Paste the content in [alert_rule.yml](./configFiles/alert_rule.yml "target=_blank")

12. Update the Prometheus configuration file
```
sudo nano /etc/prometheus/prometheus.yml
```
Copy and paste the final content of [prometheus.yml](./configFiles/prometheus.yml "target=_blank") for this documentation

13. Check the Prometheus configuration
```
promtool check config /etc/prometheus/prometheus.yml
```

14. Reload Prometheus
```
curl -X POST -u admin:devops123 http://localhost:9090/-/reload
```

## Other Operations
1. Check version

```
alertmanager --version
```

2. Check status of service
```
sudo systemctl status alertmanager
```
