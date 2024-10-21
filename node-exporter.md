# Node Exporter Installation Guide
This guide walks you through the steps for setting up `Node Exporter`, a Prometheus exporter that collects system-level metrics such as CPU, memory, and disk usage from your Linux system.

### 1. Download Node Exporter
Start by downloading the latest release of Node Exporter from the official GitHub repository. This example uses version `v1.8.2`:
```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```
**Note**: Check for the latest version in the [Node Exporter Releases](https://github.com/prometheus/node_exporter/releases)
### 2. Extract binaries
Once the file is downloaded, extract the contents using the `tar`:
```sh
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
```
Now, move the `node_exporter` binary to `/usr/local/bin`:
```sh
cd node_exporter-1.8.2.linux-amd64
sudo cp node_exporter /usr/local/bin
```
After copying the binary, clean up the extracted directory and archive to avoid clutter:
```sh
cd ..
rm ./node_exporter-1.8.2.linux-amd64.tar.gz
rm -rf ./node_exporter-1.8.2.linux-amd64
```
### 3. Create new user
For security purposes, it's a good idea to create a dedicated user for running the `node_exporter` service. The user will have no shell access and no home directory:
```sh
sudo useradd --no-create-home --shell /bin/false promuser
```
Set the ownership of the `node_exporter` binary to the newly created `promuser`:
```sh
sudo chown promuser:promuser /usr/local/bin/node_exporter
```
### 4. Setup Node Exporter as a systemd service
To manage `node_exporter` as a service, create a systemd service file:
```sh
sudo nano /etc/systemd/system/node_exporter.service
```
Add the following configuration inside the service file. This defines how `node_exporter` should run as a service.
```sh
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=promuser
Group=promuser
Type=simple
ExecStart=/usr/local/bin/node_exporter --web.listen-address=:9142
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
**Notes:**
- `ExecStart`: Specifies the command to start the Node Exporter with a custom listening address (`9142`).
- `Restart=always`: Ensures the service will be restarted if it crashes.
- `RestartSec=3`: Waits 3 seconds before attempting to restart the service.

Reload the `systemd daemon` to apply the new service:
```sh
sudo systemctl daemon-reload
```
Enable and start the `node_exporter` service:
```sh
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```
### 5. Open the port for Node Exporter
To allow `Prometheus` to scrape metrics from `Node Exporter`, you need to open the port it's listening on. Use `ufw` to allow traffic on this port:
```sh
sudo ufw allow 9142/tcp comment NODE_EXPORTER
```
### 6. Test Node Exporter
Finally, verify that `Node Exporter` is running and accessible. Use `curl` to request the metrics from the specified IP address and port (replace <ip-address> with the actual IP of your machine):
```sh
curl http://<ip-address>:9142/metrics
```
You should see a long list of metrics, similar to the following output:
```sh
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.7735e-05
go_gc_duration_seconds{quantile="0.25"} 7.3835e-05
go_gc_duration_seconds{quantile="0.5"} 9.2878e-05
go_gc_duration_seconds{quantile="0.75"} 9.9374e-05
go_gc_duration_seconds{quantile="1"} 0.000184628
go_gc_duration_seconds_sum 0.006462855
go_gc_duration_seconds_count 74
...
...
```
If the output is as expected, your `Node Exporter` is successfully installed and running.
If the output is as expected, your Node Exporter is successfully installed and running!

