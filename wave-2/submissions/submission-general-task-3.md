# Story Validator Grafana Dashboard By DeSpread

**- Grafana Dashboard**: https://story-dashboard.shachopra.com/public-dashboards/64b43909c39142b2afbe4951f8cfb93a

**- This is public dashboard from my server and does not require login**

You can change the time range by clicking the Time Range button in the top right of website.

## Setting Up Grafana Dashboard for Node Monitoring
### 1. Install docker
- Official website: https://docs.docker.com/engine/install/ubuntu/

### 2. Install Node Exporter

### 3. Configure prometheus configuration

```bash
vi prometheus.yml
```

### 4. Configure docker-compose.yaml

- Create Docker Compose Configuration for Grafana: docker-compose-grafana.yml

```bash
mkdir $HOME/docker
mkdir $HOME/docker/grafana

cd $HOME/docker/grafana
vi .env
```
```dotenv
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=admin
```

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
      - targets: []
      scheme: http
      timeout: 10s
      api_version: v1
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "story"
    static_configs:
      - targets: ["localhost:47660"]
  - job_name: "exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

```bash
vi docker-compose.yaml
```
```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:11.2.2
    container_name: grafana
    restart: unless-stopped
    ports:
      - '3000:3000'
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=${GF_SECURITY_ADMIN_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GF_SECURITY_ADMIN_PASSWORD}
    networks:
      - shared

  prometheus:
    image: prom/prometheus:v2.54.1
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - '9090:9090'
    restart: unless-stopped
    volumes:
      - ./prometheus.yaml:/etc/prometheus/prometheus.yaml
      - prometheus-storage:/prometheus
    networks:
      - shared

volumes:
  grafana-storage: {}
  prometheus-storage: {}

networks:
  shared:
    name: grafana_network
    ipam:
      config:
        - subnet: 172.16.1.0/24
```
```
docker-compose up -d
```
- This Docker Compose setup deploys Grafana in a Docker container
- Access Grafana by navigating to http://your-ip:3000

### 3. Install Prometheus

- Create Docker Compose Configuration for Prometheus: docker-compose-prometheus.yml

```
cd $HOME
mkdir docker/prometheus

cd $HOME/docker/prometheus
```
```
nano docker-compose-prometheus.yml
```
```bash
services:
  prometheus:
    image: prom/prometheus
    container_name: prom
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - 9092:9090
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prom_data:/prometheus
volumes:
  prom_data:
```
- Create Prometheus Configuration File: prometheus.yml
```
cd $HOME
mkdir docker/prometheus/prometheus
cd $HOME/docker/prometheus/prometheus
```
```
nano prometheus.yml
```
```bash
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
      - targets: []
      scheme: http
      timeout: 10s
      api_version: v1
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["your-ip:9092"]
  - job_name: "story"
    static_configs:
      - targets: ["<your-ip>:47660"]
  - job_name: "exporter"
    static_configs:
      - targets: ["<your-ip>:9100"]
```
```
cd $HOME/docker/prometheus
```
```
docker-compose -f docker-compose-prometheus.yml up -d
```
- This Docker Compose setup deploys Prometheus in a Docker container
- If prometheus is successfully setup, then you can access http://your-ip:9092

### 4. Configure story config.toml
```
nano $HOME/.story/story/config/config.toml
```
```
#make sure prometheus is set to true
prometheus = true
prometheus_listen_addr = 47660
```
### 5. Install Node Exporter
- Download Node Exporter
```
cd $HOME
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64
```
- Move the Node Exporter Binary
```
sudo cp node_exporter /usr/local/bin
```
- Create a Node Exporter User
```
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
- Configure the Service
```
sudo nano /etc/systemd/system/node_exporter.service
```
```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
- Start the service
```
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter.service
```
- If node exporter is successfully setup, then you can access http://your-ip:9100

### 6. Configuring and Using Grafana
Configure Grafana through its web interface (http://your-ip:3000) to connect to your data sources & create dashboards.

- Go to http://your-ip:3000 and click on data sources in Connections menu.
- Choose **Prometheus** as data source type
- Add Prometheus server URL: http://your-ip:9092
- Then click Save & test.
- Then go to Dashboards & click on new dashboard.
- Then click on Import dashboard.
- Upload dashboard JSON file: [Reference JSON file](https://raw.githubusercontent.com/shachopra-ai/story-grafana/refs/heads/main/grafana_dashboard.json)

Thank you!