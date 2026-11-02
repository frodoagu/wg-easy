---
title: Prometheus
---

To monitor the WireGuard server, you can use [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/). The container exposes a `/metrics/prometheus` endpoint that can be scraped by Prometheus.

## Enable Prometheus

To enable Prometheus metrics, go to Admin Panel > General and enable Prometheus.

You can optionally set a Bearer Password for the metrics endpoints. This is useful if you want to expose the metrics endpoint to the internet.

## Configure Prometheus

You need to add a scrape config to your Prometheus configuration file. Here is an example:

```yaml
scrape_configs:
    - job_name: 'wg-easy'
      scrape_interval: 30s
      metrics_path: /metrics/prometheus
      static_configs:
          - targets:
                - 'localhost:51821'
      authorization:
          type: Bearer
          credentials: 'SuperSecurePassword'
```

## Metrics

| Metric                               | Type    | Labels                                                                             | Description                                                |
| ------------------------------------ | ------- | ---------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| `wireguard_configured_peers`         | gauge   | `interface`                                                                        | Number of clients                                          |
| `wireguard_enabled_peers`            | gauge   | `interface`                                                                        | Number of enabled clients                                  |
| `wireguard_connected_peers`          | gauge   | `interface`                                                                        | Number of clients with a handshake in the last 10 minutes  |
| `wireguard_sent_bytes`               | counter | `interface`, `enabled`, `ipv4Address`, `ipv6Address`, `name`, `owner`, `ownerName` | Bytes sent to the client                                   |
| `wireguard_received_bytes`           | counter | `interface`, `enabled`, `ipv4Address`, `ipv6Address`, `name`, `owner`, `ownerName` | Bytes received from the client                             |
| `wireguard_latest_handshake_seconds` | gauge   | `interface`, `enabled`, `ipv4Address`, `ipv6Address`, `name`, `owner`, `ownerName` | Seconds since the last handshake; `0` when there is none   |
| `wireguard_peer_endpoint_info`       | gauge   | `interface`, `ipv4Address`, `name`, `owner`, `ownerName`, `endpoint`               | Always `1`; only present for clients with a known endpoint |

- `name` is the client's name. Clients can be renamed, so use `ipv4Address` to identify a client over time.
- `owner` is the username of the user that owns the client. Usernames cannot be changed, so use it to group or filter clients by user, e.g. `sum by (owner) (rate(wireguard_sent_bytes[5m]))`.
- `ownerName` is the owner's display name, or their username when no name is set. Users can change their name in their account settings, so only use it for display.

/// note | Changed labels

The `owner` and `ownerName` labels, and the `ipv4Address` label of `wireguard_peer_endpoint_info`, were added together with per-user clients. Prometheus treats series with new labels as new series, so `rate()` and alerts start over after upgrading.

///

## Grafana Dashboard

You can use the following Grafana dashboard to visualize the metrics:

[![Grafana Dashboard](https://grafana.com/api/dashboards/21733/images/16863/image)](https://grafana.com/grafana/dashboards/21733-wireguard/)

[21733](https://grafana.com/grafana/dashboards/21733-wireguard/)

/// note | Unofficial

The Grafana dashboard is not official and is not maintained by the `wg-easy` team. If you have any issues with the dashboard, please contact the author of the dashboard. It does not use the `owner` labels; add a `sum by (owner)` panel to see traffic per user.
See [#1299](https://github.com/wg-easy/wg-easy/pull/1299) for more information.

///
