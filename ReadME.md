# Monitoring Stack with Docker Compose

This project sets up a monitoring stack using Docker Compose, consisting of:

- **Grafana**: For data visualization and dashboarding.
- **Loki**: For log aggregation.
- **Prometheus**: For metrics collection and alerting.
- **Alertmanager**: For managing alerts generated by Prometheus.
- **Blackbox Exporter**: For probing endpoints and testing network connectivity.

## Prerequisites

- Docker and Docker Compose installed on your system.
- Basic understanding of Docker and monitoring concepts.
- install loki plugin for log collection from container 
   ```
      docker plugin install grafana/loki-docker-driver:2.9.2 --alias loki --grant-all-permissions
   ```

## Configuration Files

### Docker Compose File

The `docker-compose.yaml` file defines the services:

- **Loki**: Log aggregation service with a volume to store logs.
- **Alertmanager**: Sends notifications based on Prometheus alerts.
- **Prometheus**: Scrapes metrics and generates alerts.
- **Blackbox Exporter**: Probes endpoints for network connectivity checks.
- **Grafana**: Visualizes metrics and logs with dashboards.

### Prometheus Configuration (`prometheus.yaml`)

- Sets global scrape interval to 15 seconds.
- Includes `blackbox` and `prometheus` jobs.
- Configures alerting rules using `alert_rules.yml`.

### Alert Rules (`alert_rules.yml`)

Contains an alert rule for HTTP status code mismatch:

```yaml
groups:
  - name: http-probe-alerts
    rules:
      - alert: HTTPStatusCodeMismatch
        expr: probe_http_status_code{instance="Url"} != 200
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "HTTP Status Code Alert for {{ $labels.instance }}"
          description: "The HTTP status code for the instance {{ $labels.instance }} is not 200. Current status code: {{ $value }}"
```

### Alertmanager Configuration (`alertmanager.yml`)

Defines alert routing and notification settings:

- **Route**: Groups alerts and defines intervals.
- **Receiver**: Sends alerts via email using Gmail's SMTP server.

## Usage

1. Clone the repository and navigate to the project directory.

```bash
git clone <repository_url>
cd <repository_folder>
```

2. Start the stack using Docker Compose:

```bash
docker-compose up -d
```

3. Access the services in your browser:

- **Grafana**: [http://localhost:3200](http://localhost:3200)
- **Prometheus**: [http://localhost:9099](http://localhost:9099)
- **Alertmanager**: [http://localhost:9094](http://localhost:9094)
- **Blackbox Exporter**: [http://localhost:9115](http://localhost:9115)

4. Configure data sources in Grafana for Prometheus and Loki.

## Notes

- **Loki Configuration**: Ensure the `loki-url` in `x-logging` matches your setup.
- **Email Alerts**: Replace placeholders in `alertmanager.yml` with your email credentials and settings.
- **Volumes**: Persistent storage is used for Loki and Prometheus data. Ensure directories like `./prometheus-data` and `./loki-storage` exist.

## Customization

- Update `alert_rules.yml` and `alertmanager.yml` to add more alerts and customize notifications.
- Add more Prometheus scrape jobs in `prometheus.yaml` as needed.
- Modify the network and volume configurations in `docker-compose.yaml` to suit your environment.

## Stopping the Stack

To stop and remove the containers, run:

```bash
docker-compose down
```

## Troubleshooting

- Check container logs for debugging:

```bash
docker logs <container_name>
```

- Ensure all required ports are available and not blocked by a firewall.
