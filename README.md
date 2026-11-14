# Network monitoring stack

Prometheus collects SNMP data and Grafana provides dashboards. The `SIT Event APs (Aruba controller)` dashboard selects APs by the group name reported by the Aruba controller, so group changes are reflected automatically.

## Setup

1. Copy `.env.example` to `.env` and configure the Grafana credentials, `Prometheus_ADMIN_USER`, and `PROMETHEUS_BASIC_PASSWORD`. The latter is the plain Prometheus password used by Grafana; keep it out of Git.
2. Create `prometheus/web.local.yml` with a `basic_auth_users` mapping containing the Prometheus username and a bcrypt hash of its password. Create `prometheus/prometheus.password` containing the same plain password, without a trailing newline, for Prometheus to scrape itself. The username in the `prometheus` scrape job in `prometheus/prometheus.yml` must match the web username. Changing these credentials requires updating all three locations and the `.env` settings.
3. Copy `snmp/auth.yml` to `snmp/auth.local.yml` and configure `switch_v2` for switches and `ap_v2` for the Aruba controller. Configure SNMPv3 instead if applicable. Never put real credentials in tracked files: `snmp/aruba.yml` contains only the metric definitions. After changing credentials on a running stack, run `docker compose restart snmp-exporter`.
4. Create `prometheus/targets/aruba-controller.local.yml` with the controller IP as a file discovery target. The controller must be reachable by the exporter over UDP 161 and allow the Docker host's source address. Its SNMP community must permit the Aruba MIB.
5. Start Docker Desktop, then run `docker compose up -d`. Open Grafana at http://localhost:3000 and select **Network → SIT Event APs (Aruba controller)**. Prometheus is at http://localhost:9090 and requires its configured login.

`.env`, `snmp/*.local.yml`, `prometheus/*.local.yml`, `prometheus/targets/*.local.yml`, and `prometheus/*.password` are Git-ignored. Ignore rules do not protect files already tracked by Git; check `git status` before committing. Never commit credentials.

## Interpreting the dashboard

The controller reports AP status and group membership; the controller SNMP panel separately shows whether the scrape succeeded. Client counts sum the radios on each AP. RX and TX show rates of BSSID data counters, summed per AP; the separate bandwidth gauge is reported by the controller. Traffic direction is relative to the AP. Uptime is the time since the AP last bootstrapped with the controller. A failed controller scrape prevents AP data from refreshing; the dashboard is not a ping test.

The original overview also contains ping and alert panels for components that are currently disabled, so those panels do not represent live device status. Alertmanager is not enabled.

## EventOps VM dashboard exports

`grafana/dashboard-exports/` contains snapshots of the existing EventOps overview and SIT Event AP dashboards for manual import into Grafana. They are not automatically provisioned; their queries depend on the EventOps VM datasource and SNMP configuration. The client scrape is restricted to the eight AP names listed in `prometheus/prometheus.yml`; update that list if the monitored group changes.

For the `event_ap_switches` job, provide `prometheus/targets/event-switches.local.yml` in Prometheus file discovery format.
