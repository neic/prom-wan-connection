# Prometheus WAN connection

This pings three cloud providers continuously to calculates the uptime of the
WAN connection. It displays the resulting uptime in a graph.

## Getting started

Make sure you have Docker and docker-compose installed. On macOS, just install
Docker Desktop.

- Clone this repo.
- In the resulting folder, run `docker-compose up -d`. This will download all
  the images and start them in the background.
- Open your web browser at `http://localhost:3000` to access Grafana.
- Login with `admin:admin` and change the password.
- I the upper left corner, click the breadcrubs and select "Auto provisioned" -> "Network".

You stop all the services with `docker-compose down`. If you also want to delete
persistent data run `docker-compose down -v` to delete volumes as well.

If you want to see the log output from the services running in the background
run `docker-compose logs <service-name>`.

## Inner workings

This is done by three microservices: prometheus-blackbox-exporter, prometheus
and grafana.

### Prometheus-blackbox-exporter

Probes various locations over different protocols. In
`config/prometheus/blackbox.yml` we define a single proper: the ICMP prober.
ICMP is the protocol responsible for the normal `ping** requests.

### Prometheus

An time-series database for metrics. It requests metrics from targets define in
`config/prometheus/prometheus.yml`. There is a single job defined called
`pings`. It connects to the blackbox-exporter and tells it which targets to ping
(Google, GitHub and Cloudflare). It also sets the frequency to which to do this:
every 5 seconds.

The `config/prometheus/rules.yml` tells prometheus to calculate some new metrics
based on the above pings. The new metrics are:
 - `wanUptime_connected`: True if at least two cloud providers are up.
 - `wanUptime_slidingWindow_<time>`: The averange uptime over the last `<time>`.
 
### Grafana

Displays the resulting data as a dashboard. The
`config/grafana/provisioning/datasources/datasources.yml` tells Grafana to use
prometheus as a datasource. The
`config/grafana/provisioning/dashboards/dashboards.yml` tells Grafana to
autoload dashboards defined in `config/grafana/dashboards/`. There is a single
dashboard `network.json`. It defines four panels:
- Three uptime stats for the last 15min, 1h and 24h.
- A graph that shows missed pings as bars and the sliding uptime stats for the
  selected time range.

