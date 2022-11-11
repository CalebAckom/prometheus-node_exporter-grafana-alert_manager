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
