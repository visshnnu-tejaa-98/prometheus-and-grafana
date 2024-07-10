# Prometheus and Grafana

Install Prometheus and Grafana on a Linux EC2 machine, connect Prometheus to Grafana, and create a dashboard to view metrics.

## Step 1: Prometheus Installation

1.  Create an EC2 server and connect the server using mobaxterm

    ![alt image](/images/prometheus-server.png)

2.  Create a system user

    ```bash
    # To Create a system user
    sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false prometheus

    ```

3.  Download prometheus using wget

    ```bash
    wget https://github.com/prometheus/prometheus/releases/download/v2.45.0-rc.1/prometheus-2.45.0-rc.1.linux-amd64.tar.gz
    ```

4.  Untar the downloaded prometheus file

    ```bash
    tar -xvf prometheus-2.45.0-rc.1.linux-amd64.tar.gz
    ```

5.  Create a new directory called prometheus under /etc/

    ```bash
    sudo mkdir -p /data /etc/prometheus
    ```

6.  Move to prometheus directory

    ```bash
    cd prometheus-2.45.0-rc.1.linux-amd64/
    ```

7.  Move some files to /bin location

    ```bash
    sudo mv prometheus promtool /usr/local/bin/
    ```

8.  Moving consoles directory also

    ```bash
    sudo mv consoles/ console_libraries/ /etc/prometheus/
    ```

9.  Moving prometheus configuration file

    ```bash
    sudo mv prometheus.yml /etc/prometheus/prometheus.yml
    ```

10. Changing ower of the directory to prometheus

    ```bash
    sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
    ```

11. Remove unnecessary directoryies and files

    ```bash
    cd
    rm -rf prometheus\*

    ```

12. Check the prometheus version

    ```bash
    prometheus –version
    ```

13. Create a new file called prometheus.service

    ```bash
    sudo vim /etc/systemd/system/prometheus.service
    ```

14. Add the following data

    ```bash
    [Unit]

    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/data \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle

    [Install]
    WantedBy=multi-user.target

    ```

15. Start the service by using following commands

    ```bash
    # To enable service
    sudo systemctl enable prometheus
    # To Start service
    sudo systemctl start prometheus
    # To check status of a service
    sudo systemctl status prometheus
    ```

16. Copy the public ip and and navigate to this URL in browser
    http://<public_ip>:9090

## Step 2: Node Exporter Installation

1.  Create a EC2 instance and connect it to mobaxterm

    ![alt image](/images/capstone-project-server.png)

2.  Creating a system user

    ```bash
    sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter

    ```

3.  Downloading the node exporter

    ```bash
    wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
    ```

4.  Untar the downloaded file
    ```bash
    tar -xvf node_exporter-1.6.0.linux-amd64.tar.gz
    ```
5.  Move binaries to /usr/local/bin location

    ```bash
    sudo mv \
    node_exporter-1.6.0.linux-amd64/node_exporter \
    /usr/local/bin/

    ```

6.  cleaning the un necessary files

    ```bash
    rm -rf node_exporter*
    ```

7.  Cheking the node_exporter version

    ```bash
    node_exporter –version
    ```

8.  Creating a node_exporter.service file

    ```bash
    sudo vim /etc/systemd/system/node_exporter.service
    ```

9.  Add the following code in to node_exporter.yaml file

    ```bash
    [Unit]

    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/node_exporter \
    --collector.logind

    [Install]
    WantedBy=multi-user.target
    ```

10. Start the node_exporter using the following commands

    ```bash
    # To enable the node_exporter
    sudo systemctl enable node_exporter
    # To start the node_exporter
    sudo systemctl start node_exporter
    # To check status of the node_exporter
    sudo systemctl status node_exporter
    ```

## Step 3: node_exporter configuration in prometheus server

1. Go to prometheus server and open the prometheus.yml file

   ```bash
   sudo vim /etc/prometheus/prometheus.yml
   ```

2. Add the following code in scrap_configs section

   ```bash
     - job_name: node_export
       static_configs:
         - targets: ["<node_exported_public_ip>:9100"] # <node_exported_public_ip>:9100

   ```

3. Check the syntax error using promtool

   ```bash
   promtool check config /etc/prometheus/prometheus.yml
   ```

4. Reload and restart the prometheus service

   ```bash
   systemctl daemon-reload
   systemctl restart prometheus
   systemctl status prometheus
   ```

5. Reload the prometheus dashboard in browser, you will able to see the node_exporter in targets

   ![alt image](/images/prometheus-dashboard.png)

## Step 4: Grafana installation in prometheus server

1. Go to prometheus server and install the grafana

   ```bash
   sudo apt-get install -y apt-transport-https software-properties-common
   ```

2. Adding GPG key

   ```bahs
   wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
   ```

3. Add this repository for stable releases.

   ```bash
   echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
   ```

4. Updating the server

   ```bash
   sudo apt-get update
   ```

5. Installing Grafana

   ```bash
   sudo apt-get -y install grafana
   ```

6. Starting the grafana service

   ```bash
   sudo systemctl enable grafana-server
   sudo systemctl start grafana-server
   sudo systemctl status grafana-server
   ```

7. Navigate to http://<ip>:3000 in the browser to see the grafana login page

8. Login using the default creds
   username: admin
   password: admin

9. click on datasources in grafana home page > prometheus > give localhost:9090 > save and test

**Creating as dashboard**

1. click on dashboards in side bar
2. create dashboard > import dashboard > give id as 1860 (node exporter)
3. click on load it > give prometheus > import
4. Now, you will be able to see the grafana dashboard

   ![alt image](/images/grafana-dashboard.png)
