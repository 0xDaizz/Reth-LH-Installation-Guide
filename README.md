# Reth-LH-Installation-Guide
How to Build Reth + Lighthouse + Prometheus + Grafana

## 0. Spec & Env.
- CPU : AMD Ryzen 5 5600G
- SSD : Samsung 970 EVO Plus 2TB * 2 = 4TB by mdadm RAID0 of Ubuntu
- RAM : DDR4 16 * 2 = 32GB

- OS : Ubuntu 22.04 LTS

---
## 1. Prometheus & Grafana

### 1-1. Prometheus
- installation
```
sudo apt update
sudo apt install prometheus
```

- edit prometheus.yml
Directory: ```/etc/prometheus/prometheus.yml```

Replace .yml with [this](https://github.com/paradigmxyz/reth/blob/main/etc/prometheus/prometheus.yml)

- run
```
sudo systemctl start prometheus

# automatically run on log-on
sudo systemctl enable prometheus
```

### 1-2. Grafana
- installation
ref: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-grafana-on-ubuntu-22-04)
```
wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana
sudo systemctl start grafana-server

# automatically run on log-on
sudo systemctl enable grafana-server
```

- add dashboard
1. enter http://localhost:3000 - log in admin/admin
2. left toggle menu button - administration - Data Sources - Add new data source - Prometheus
3. Fill HTTP - Prometheus server URL with http://localhost:9090
4. click Save & test
5. left toggle menu button - Dashboards - New - Import
6. Upload JSON file from [here](https://github.com/paradigmxyz/reth/blob/main/etc/grafana/dashboards/overview.json)
7. click Load
