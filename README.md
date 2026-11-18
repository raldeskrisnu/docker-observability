# Observability Stack (Local)

Stack monitoring/observability untuk development lokal: **Grafana**, **Prometheus**, **Loki**, **Jaeger**.

## Services

| Service | URL | Fungsi |
|---|---|---|
| Grafana | http://localhost:3001 | Dashboard & visualisasi (anonymous admin, no login) |
| Prometheus | http://localhost:9090 | Metrics storage & query |
| Loki | http://localhost:3100 | Log aggregation |
| Jaeger UI | http://localhost:16686 | Distributed tracing |

Semua datasource (Prometheus, Loki, Jaeger) sudah auto-provisioned di Grafana — langsung kepakai tanpa setup manual.

## Cara Pakai

```bash
cd ~/observability

docker compose up -d       # start semua service
docker compose ps          # cek status
docker compose logs -f     # tail log semua container
docker compose down        # stop (data tetap)
docker compose down -v     # stop + hapus semua data (volumes)
```

## Kirim Data ke Stack

### Metrics → Prometheus
Tambahkan target scrape di `prometheus.yml`:
```yaml
scrape_configs:
  - job_name: my-service
    static_configs:
      - targets: ["host.docker.internal:8080"]  # app yang jalan di host
```
Lalu `docker compose restart prometheus`. App harus expose endpoint `/metrics` (format Prometheus).

### Traces → Jaeger
Kirim OTLP ke:
- gRPC: `localhost:4317`
- HTTP: `localhost:4318`

Contoh env var untuk SDK OpenTelemetry:
```
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

### Logs → Loki
Loki kosong by default — perlu agent log shipper (misal Grafana Alloy atau Promtail) yang push ke `http://localhost:3100/loki/api/v1/push`. Belum termasuk di stack ini.

## File

| File | Isi |
|---|---|
| `docker-compose.yml` | Definisi 4 container + volumes |
| `prometheus.yml` | Scrape config Prometheus |
| `grafana-datasources.yml` | Auto-provision datasource Grafana |

## Known Limitations

- Tidak ada auth di Grafana (anonymous admin) — local only, jangan expose ke network lain.
- Loki tidak nerima log otomatis — butuh log shipper terpisah.
- Data tersimpan di Docker named volumes (`observability_*-data`), persist antar restart kecuali pakai `down -v`.

<!-- ponytail: no log shipper, no auth, single-node. Upgrade: tambah Alloy/Promtail service kalau butuh log otomatis dari app. -->
