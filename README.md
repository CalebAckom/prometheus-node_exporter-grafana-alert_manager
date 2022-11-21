# Prometheus NodeExporter Grafana Alertmanager Setup
This is a documentation on setting up Prometheus Node Exporter Grafana and Alertmanager on Ubuntu 20.04

**For the following reasons, dedicated Linux users or system accounts should be created for each service**

1. To reduce the impact in a case of an incident with a service
2. To simplify administration because it is easier to track down what resources belong to which service.

## Installing Prometheus
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
