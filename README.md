# Network monitoring stack (Prometheus + Grafana)

1. `cp .env.example .env` and set a strong Grafana password.
2. Edit `prometheus/targets/*.yml` with your switch / AP / camera / NDI IPs.
3. Edit `snmp/auth.yml` with your SNMP community (or SNMPv3 user).
4. Enable SNMP on every switch/AP (allow the Docker host's IP).
5. `docker compose up -d`
6. Grafana: http://HOST:3000  - Prometheus: http://HOST:9090  - Alertmanager: http://HOST:9093
7. Reload target/rule changes without restart: `curl -X POST http://localhost:9090/-/reload`

Tip: set switch port descriptions like `NDI-cam1`, `AP-lobby`, `UPLINK-core` - dashboards and alerts key off these names.
