# Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager

## Introduction
The Enterprise Monitoring & Alerting Stack (EMAS) is a production-grade, two-server observability solution designed to provide real-time metrics, centralized log management, and proactive alerting for distributed applications.
Built on industry-leading tools — Prometheus, Grafana, Elasticsearch, Logstash, and Alertmanager — EMAS delivers a unified platform where DevOps teams can monitor system health, analyze application logs, and respond to issues before they impact users

In this project, the architecture is split into:
- Monitoring Server: Hosts the complete monitoring and alerting stack, aggregating data from application servers.
- App Server: Runs the target application, exposes system metrics via Node Exporter, and ships application logs using Filebeat.

This setup replicates real-world production environments, where monitoring infrastructure is isolated from application workloads, ensuring high availability, scalability, and operational security.

## Statement of Problem
In many production environments, system performance issues, application errors, and security incidents often go undetected until they significantly impact end users.
This is largely due to:

- Fragmented monitoring solutions where metrics and logs are stored in separate, unintegrated tools
- Lack of proactive alerting, forcing teams to react to outages instead of preventing them
- Overloaded application servers that must handle both workload processing and monitoring tasks, reducing overall performance
- Limited visibility into real-time system health, making root cause analysis slow and inefficient

Without a centralized, real-time monitoring and alerting platform, DevOps teams face delays in identifying issues, higher downtime, and increased operational costs.

## Project Objectives
- To design and deploy a two-server monitoring architecture.
- To collect real-time system metrics using Prometheus and Node Exporter.
- To centralize application and system logs with Filebeat, Logstash, and Elasticsearch.
- To integrate metrics and logs into Grafana for unified visualization.
- To configure Alertmanager for proactive notifications via email or Slack.

## Tools & Technologies
- Prometheus – Open-source monitoring system that collects and stores time-series metrics.
- Grafana – Visualization and dashboard platform for metrics, logs, and alerts.
- Elasticsearch – Distributed search and analytics engine used for storing and querying logs.
- Logstash – Data processing pipeline that ingests logs from Filebeat and sends them to Elasticsearch.
- Filebeat – Lightweight log shipper that forwards logs from the App Server to Logstash.
- Alertmanager – Component of Prometheus that manages alerts and sends notifications to channels like email or Slack.
- Node Exporter – Prometheus exporter that collects system metrics (CPU, memory, disk) from Linux servers.

## Project Steps
i.	Set up Monitoring Server (EC2)

ii.	Install Node Exporter on App Server

iii.	Configure Prometheus on Monitoring Server

iv.	Create Metrics Dashboards in Grafana

v.	Configure Log Pipeline (App → Monitoring)

vi.	Set Up Alerting

vii.	Build Master Overview Dashboard


NOTE: We already have a full-stack application (frontend + backend), fully Dockerized and running on an EC2 instance. Before starting the monitoring project, ensure your own application is running in a similar setup, then provision a separate Monitoring Server (minimum t3.medium, 4 GB RAM, 50 GB disk) to host Prometheus, Grafana, Elasticsearch, Logstash, and Alertmanager.

## Project Implementation

### Step 1: Set up Monitoring Server (EC2)

#### Setting Up the Prometheus User and Directories

First, we will create a dedicated user for Prometheus and the necessary directories to store its configuration and data. This is a security best practice.

- Create a new system user named prometheus. This command creates a user that can't log in directly, which is more secure.
  ```bash
  sudo useradd --no-create-home --shell /bin/false prometheus
	```

- Create the directory where Prometheus configuration files will live.
  ```bash
  sudo mkdir /etc/prometheus
  ```

- Create the directory where Prometheus will store its data.
  ```bash
  sudo mkdir /var/lib/prometheus
  ```

- Set the owner of the data directory to the prometheus user.
  ```bash
  sudo chown prometheus:prometheus /var/lib/prometheus
  ```

#### Downloading and Moving Prometheus Files
- Download the latest stable version of Prometheus. We'll use wget for this.
  ```bash
  wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
  ```
  <img width="975" height="292" alt="image" src="https://github.com/user-attachments/assets/0430a0ff-92e5-4bcd-9efa-0ab078ce5765" />

- Extract the files from the downloaded tarball.
  ```bash
  tar xvfz prometheus-2.46.0.linux-amd64.tar.gz
  ```

  <img width="975" height="381" alt="image" src="https://github.com/user-attachments/assets/d8617c36-899b-4bbe-970b-af7fcbccf1c6" />


- This creates a new directory. Navigate into it.
  ```bash
  cd prometheus-2.46.0.linux-amd64
  ```
- Copy the prometheus and promtool binaries to the /usr/local/bin/ directory, which is in your system's PATH.
  ```bash
  sudo mv prometheus promtool /usr/local/bin/
  ```

  - Set the owner of the prometheus and promtool binaries to the prometheus user.
  ```bash
  sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
  ```

  - Copy the consoles and console_libraries directories to the /etc/prometheus directory.
  ```bash
  sudo mv consoles console_libraries /etc/prometheus/
  ```

  - Set the owner of the consoles directories and all their files to the prometheus user.
  ```bash
  sudo chown -R prometheus:prometheus /etc/prometheus/consoles /etc/prometheus/console_libraries
  ```

#### Creating the Prometheus Configuration File

Now we need to create the main configuration file, prometheus.yml. This file tells Prometheus what to monitor. We'll start with a basic configuration that tells Prometheus to monitor itself.

- Use the vi text editor to create and open the configuration file.
  ```bash
  sudo vi /etc/prometheus/prometheus.yml
  ```
  - Paste the following content into the editor. This is a basic but complete configuration.
    
[Prometheus Configuration](https://github.com/Ogbunugafor-Philip/Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager/blob/main/Prometheus%20Configuration)


<img width="869" height="269" alt="image" src="https://github.com/user-attachments/assets/97955ce1-d72e-4034-b2a4-963b3a6f6758" />

    
 
#### What this configuration does:
- global: Defines a default scrape interval of 15 seconds.
- scrape_configs: This is a list of all the services Prometheus will monitor.
- - job_name: 'prometheus': This creates a monitoring job named 'prometheus'.
- targets: ['localhost:9090']: This tells Prometheus to scrape metrics from itself, at the default port 9090.

#### Creating the Prometheus Systemd Service

To run Prometheus as a background service, we will create a systemd service file. This allows you to manage it easily with commands like sudo systemctl start prometheus.
- Use vi to create and open the service file.
```bash
sudo vi /etc/systemd/system/prometheus.service
```

- Paste the following content into the editor. This tells systemd how to run Prometheus.
  
[Prometheus systemd service](https://github.com/Ogbunugafor-Philip/Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager/blob/main/prometheus%20systemd%20service)

<img width="975" height="259" alt="image" src="https://github.com/user-attachments/assets/a268665c-a550-4565-b660-516a96057758" />

#### What this configuration does:
- [Unit]: Describes the service and its dependencies.
- [Service]: Specifies how to run the service. User and Group ensure it runs as the user we created. ExecStart is the main command that starts Prometheus, pointing to the           configuration file, data storage, and console files.
- [Install]: Tells systemd to start the service when the system boots up.

#### Starting and verifying the Prometheus Service

- These final commands will start Prometheus and ensure it's running correctly. Reload systemd to recognize the new service file.
```bash
sudo systemctl daemon-reload
```

- Start the Prometheus service.
```bash
sudo systemctl start prometheus
```

- Enable the service to start automatically on system boot.
```bash
sudo systemctl enable prometheus
```

- Check the status of the Prometheus service to ensure it is running without errors.
```bash
sudo systemctl status prometheus
``` 
<img width="975" height="447" alt="image" src="https://github.com/user-attachments/assets/30e25ca0-4a20-4f26-b894-4977ca20d361" />

What to look for: The output should show a green dot and the text Active: active (running). If you see an error, please let me know, and we can troubleshoot it.
Once you have confirmed that the service is active and running, you can open your web browser and navigate to http://<Your-Monitoring-Server-IP>:9090. (make sure to open port 9000 on your monitoring server). You should see the Prometheus web interface.
 <img width="975" height="344" alt="image" src="https://github.com/user-attachments/assets/8d5f88bf-3767-4b13-8eed-5177739b3343" />


#### Installing Grafana
- We need to install some packages that allow us to get a repository key and manage repositories over HTTPS.
```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
``` 
<img width="975" height="504" alt="image" src="https://github.com/user-attachments/assets/b4468a2f-5962-40fb-b599-3fc4c6a5e335" />

- Now, we will download and add the Grafana repository key. This key is used to verify that the software you're downloading is authentic.
```bash
sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

- Next, we will add the Grafana repository to your system's apt sources list. This tells your system where to find the Grafana software.
```bash
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
``` 
<img width="975" height="404" alt="image" src="https://github.com/user-attachments/assets/aa06b6a0-0ae9-4b10-9c01-5f06293e369e" />

- Update your local package list to include the new Grafana repository.
```bash
sudo apt-get update
```

- Finally, install Grafana from the repository.
```bash
sudo apt-get install -y grafana
 ```
<img width="975" height="387" alt="image" src="https://github.com/user-attachments/assets/f4144789-c6f1-4705-a7e1-86f2f38e734f" />

#### Starting and Verifying the Grafana Service
Now, we will start the service and make sure it is running properly.

- Reload systemd to recognize the new Grafana service.
```bash
sudo systemctl daemon-reload
```

- Start the Grafana service.
```bash
sudo systemctl start grafana-server
```
- Enable the service to start automatically on system boot
```bash
sudo systemctl enable grafana-server
```

- Check the status of the Grafana service to ensure it is running without errors.
```bash
sudo systemctl status grafana-server
```
<img width="975" height="239" alt="image" src="https://github.com/user-attachments/assets/b08466b5-1efb-4bc3-a8e0-bb7d66e87eb6" />

What to look for: The output should show a green dot and the text Active: active (running

Once you have confirmed that the service is active, you can open your web browser and navigate to http://<Your-Monitoring-Server-IP>:3000. (make sure to open port 3000 on your monitoring server)

- The default username and password are admin and admin.
- Grafana will ask you to create a new password on your first login.
<img width="975" height="369" alt="image" src="https://github.com/user-attachments/assets/26ca2840-8c44-42e9-b86c-3bc77b603eac" />
 

#### Installing Elasticsearch
- We will add the GPG key for the Elastic repository. This key is used to verify the authenticity of the Elasticsearch package.
```bash
sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

- Now, we'll save the repository definition to the /etc/apt/sources.list.d/ directory.
```bash
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

- Update your local package list to include the new Elastic repository.
```bash
sudo apt-get update
```

- Finally, install Elasticsearch from the repository.
```bash
sudo apt-get install -y elasticsearch
``` 
<img width="975" height="225" alt="image" src="https://github.com/user-attachments/assets/315f54f6-d615-4922-a56f-aee511e704bf" />

#### Configuring and Starting Elasticsearch
- We need to open the Elasticsearch configuration file. Remember to use vi as you requested.
```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```

Once vi is open, press i to enter insert mode. Then, find the network.host and http.port settings and make them look like this.
Note: You need to replace your_monitoring_server_private_ip with the actual private IP of your Monitoring Server. If you don't know it, you can find it by running the command hostname -I in a new terminal window.

[Elasticsearch Configuration](https://github.com/Ogbunugafor-Philip/Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager/blob/main/elasticsearch%20configuration

<img width="975" height="298" alt="image" src="https://github.com/user-attachments/assets/8a00ba10-4f57-491d-ab48-ab894c477295" />


 
Once you have added/edited those lines, press the Esc key to exit insert mode, then type: wq and press Enter to save and quit.

#### Starting and Verifying Elasticsearch
- Reload systemd to recognize the Elasticsearch service.
```bash
sudo systemctl daemon-reload
```

- Start the Elasticsearch service.
```bash
sudo systemctl start elasticsearch
```

- Enable the service to start automatically on system boot.
```bash
sudo systemctl enable elasticsearch
```

- Check the status of the service to ensure it is running.
```bash
sudo systemctl status elasticsearch
``` 
<img width="975" height="305" alt="image" src="https://github.com/user-attachments/assets/ba20bc64-e9c8-4b1c-a0ec-d951db988ca2" />

What to look for: The output should show a green dot and the text Active: active (running).

- Now, let's verify that Elasticsearch is responding. Run this curl command.
```bash
curl -X GET 'http://<monitoring server private ip>:9200'
```
<img width="975" height="361" alt="image" src="https://github.com/user-attachments/assets/464622b5-583f-48b9-af76-666547504cc0" />
 
If successful, this command should return a JSON response with details about your cluster.


#### Installing Logstash
- First, update your local package list to ensure you have the latest information from the repository.
```bash
sudo apt-get update
```

- Now, install Logstash.
```bash
sudo apt-get install -y logstash
``` 
<img width="975" height="258" alt="image" src="https://github.com/user-attachments/assets/d887f4ee-af88-493b-aba4-50ff40b1bf38" />

#### Configuring the Logstash Pipeline
Logstash needs a configuration file to tell it what to do. This file defines the input (where logs come from) and the output (where to send them).
We will create a configuration file that listens for logs from your App Server (via Filebeat) and forwards them to Elasticsearch.

- Open a new configuration file for Logstash. Remember to use vi.
```bash
sudo vi /etc/logstash/conf.d/02-beats-output.conf
```
Press i to enter insert mode, and paste the following content into the file.

Note: The hosts line is set to the private IP address of your Monitoring Server, which is where Elasticsearch is running.

[Logstash Configuration](https://github.com/Ogbunugafor-Philip/Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager/blob/main/logstash%20configuration)


#### Starting and Verifying the Logstash Service
- Reload systemd to recognize the Logstash service.
```bash
sudo systemctl daemon-reload
```

- Start the Logstash service.
```bash
sudo systemctl start logstash
```

- Enable the service to start automatically on system boot.
```bash
sudo systemctl enable logstash
```
- Check the status of the service to ensure it is running.
```bash
sudo systemctl status logstash
``` 
<img width="975" height="318" alt="image" src="https://github.com/user-attachments/assets/56df7a57-9d4a-4b2c-96e7-ad98b7e05de1" />

Note on Logstash status: Logstash can sometimes take a minute or two to start up. If the status shows active (running), it is working. If it shows active (exited) after a moment, this can also be normal if it has not yet received any log data. The key is that the service should not be in a failed state.

#### Setting Up the Alertmanager User and Directories
- Create a new system user named alertmanager. This command creates a user that can't log in directly, which is a good security practice.
```bash
sudo useradd --no-create-home --shell /bin/false alertmanager
```

- Create the directory where Alertmanager configuration files will live.
```bash
sudo mkdir /etc/alertmanager
```

- Create the directory where Alertmanager will store its data.
```bash
sudo mkdir /var/lib/alertmanager
```
- Set the owner of the data directory to the alertmanager user we created.
```bash
sudo chown alertmanager:alertmanager /var/lib/alertmanager
```

#### Downloading and Moving Alertmanager Files
- Download the latest stable version of Alertmanager.
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz
```
<img width="975" height="407" alt="image" src="https://github.com/user-attachments/assets/0fc1385b-fd4f-469a-b7d1-ecf7d164e2bc" />

- Extract the files from the downloaded tarball.
```bash
tar xvfz alertmanager-0.26.0.linux-amd64.tar.gz
``` 
<img width="975" height="225" alt="image" src="https://github.com/user-attachments/assets/b7f437eb-390c-4054-99f6-17db55810698" />

- This creates a new directory. Navigate into it.
```bash
cd alertmanager-0.26.0.linux-amd64
```

- Copy the alertmanager and amtool binaries to the /usr/local/bin/ directory.
```bash
sudo mv alertmanager amtool /usr/local/bin/
```
- Set the owner of the alertmanager and amtool binaries to the alertmanager user.
```bash
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager /usr/local/bin/amtool
```

#### Creating the Alertmanager Configuration File
Alertmanager needs a configuration file, alertmanager.yml, to tell it how to handle incoming alerts and where to send notifications. We'll start with a basic configuration.

- We need to open a new configuration file for Alertmanager. Remember to use vi.
```bash
sudo vi /etc/alertmanager/alertmanager.yml
```

- Press i to enter insert mode, and paste the following content into the file. This configuration sets up a basic receiver for alerts.

[Alertmanager Config](https://github.com/Ogbunugafor-Philip/Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager/blob/main/alertmanager%20config)

After pasting the content, press the Esc key to exit insert mode, then type :wq and press Enter to save and quit.


#### Creating the Alertmanager Systemd Service
- Use vi to create and open the service file.
```bash
sudo vi /etc/systemd/system/alertmanager.service
```

Press i to enter insert mode, and paste the following content into the file.

[Alertmanager systemd service](https://github.com/Ogbunugafor-Philip/Enterprise-Monitoring-Two-Server-Architecture-with-Grafana-Prometheus-Elasticsearch-and-Alertmanager/blob/main/alertmanager%20systemd%20service)


After pasting the content, press the Esc key to exit insert mode, then type :wq and press Enter to save and quit.

#### Starting and Verifying the Alertmanager Service
- Reload systemd to recognize the new service file.
```bash
sudo systemctl daemon-reload
```

- Start the Alertmanager service.
```bash
sudo systemctl start alertmanager
```

- Enable the service to start automatically on system boot.
```bash
sudo systemctl enable alertmanager
```
- Check the status of the Alertmanager service to ensure it is running without errors.
```bash
sudo systemctl status alertmanager
``` 
<img width="975" height="317" alt="image" src="https://github.com/user-attachments/assets/96036fca-a709-4f4c-86aa-d3c3de7dd2d7" />

What to look for: The output should show a green dot and the text Active: active (running).

Once you have confirmed that the service is active, you can open your web browser and navigate to http://<Your-Monitoring-Server-IP>:9093 (make sure you open port 9093 in your monitoring server security). You should see the Alertmanager web interface.
<img width="975" height="325" alt="image" src="https://github.com/user-attachments/assets/bb937ee7-33f9-42d6-9cc4-2e854dcd5849" />
 
### Step 2: Install Node Exporter on App Server
All commands would be performed on our App Server, not the Monitoring Server.
Setting Up the Node Exporter User and Directory
First, we will create a dedicated user for Node Exporter and a temporary directory to download the software.
	Create a new system user named node_exporter. This command creates a user that can't log in directly, which is a good security practice.
sudo useradd --no-create-home --shell /bin/false node_exporter

Downloading and Moving Node Exporter Files
	Download the latest stable version of Node Exporter.
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
 

	Extract the files from the downloaded tarball.
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
 

	This creates a new directory. Navigate into it.
cd node_exporter-1.7.0.linux-amd64
	Copy the node_exporter binary to the /usr/local/bin/ directory.
sudo mv node_exporter /usr/local/bin/
	Set the owner of the node_exporter binary to the node_exporter user.
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

Creating the Node Exporter Systemd Service
	We need to create a systemd service file to run Node Exporter as a background process. Use vi to create the service file.
sudo vi /etc/systemd/system/node_exporter.service
	Press i to enter insert mode, and paste the following content into the file. This configuration tells the system to run the node_exporter binary as the node_exporter user.


[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
After pasting the content, press the Esc key to exit insert mode, then type :wq and press Enter to save and quit.

Starting and Verifying the Node Exporter Service
	Reload systemd to recognize the new service file.
sudo systemctl daemon-reload
	Start the Node Exporter service.
sudo systemctl start node_exporter
	Enable the service to start automatically on system boot.
sudo systemctl enable node_exporter
	Check the status of the Node Exporter service to ensure it is running without errors.
sudo systemctl status node_exporter
 
What to look for: The output should show a green dot and the text Active: active (running).

Once you have confirmed that the service is active, you can open your web browser and navigate to http://<Your-App-Server-IP>:9100/metrics. You should see a large amount of raw metrics data.
 

Installing Docker on the App Server
	Update your local package list.
sudo apt-get update
	Install the necessary packages to allow apt to use a repository over HTTPS.
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
	Add Docker's official GPG key.
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
	Set up the Docker repository.
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	Update your package list again to include the new Docker repository.
sudo apt-get update
	Install the latest version of Docker Engine, containerd, and Docker Compose.
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
Running the cAdvisor Docker Container
We will run cAdvisor as a Docker container. The following command will download the cAdvisor image, run it in the background, and expose its metrics on port 8080.
	Run the cAdvisor container
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:latest
 

Verifying the cAdvisor Container
	Run the docker ps command to list all running containers.
sudo docker ps
What to look for: You should see an entry for a container named cadvisor with the image gcr.io/google-containers/cadvisor:latest and a status of Up.
 
Once you have confirmed that the container is running, you can access the metrics page from your browser.
Open your web browser and navigate to http://<Your-App-Server-IP>:8080. You should see a large amount of raw container metrics data.
 

Step 3: Configuring Prometheus on the Monitoring Server
We will now add two new jobs to our prometheus.yml file to scrape the Node Exporter and cAdvisor metrics from the App Server.
	On your Monitoring Server, open the Prometheus configuration file.
sudo vi /etc/prometheus/prometheus.yml
	Add the following two new jobs to the scrape_configs section. Make sure to replace <Your-App-Server-IP> with the actual public IP address of your App Server.
YAML
# Add this job to scrape Node Exporter on the App Server
- job_name: 'node_exporter_app_server'
  static_configs:
    - targets: ['<Your-App-Server-IP>:9100']

# Add this job to scrape cAdvisor on the App Server
- job_name: 'cadvisor_app_server'
  static_configs:
    - targets: ['<Your-App-Server-IP>:8080']
After you have added the new jobs, save and close the file. Press Esc, then type :wq and press Enter.
	Check the syntax of the Prometheus configuration file to ensure there are no errors.
promtool check config /etc/prometheus/prometheus.yml
 

	Restart the Prometheus service to load the new configuration.
sudo systemctl restart prometheus
After these have completed, let us navigate to our Prometheus UI in our browser at http://<Your-Monitoring-Server-IP>:9090/targets and know if we can see all three targets (prometheus, node_exporter_app_server, and cadvisor_app_server) as UP.
 

Step 4: Create Metrics Dashboards in Grafana
On your Monitoring Server, open your web browser and navigate to the Grafana dashboard at http://<Your-Monitoring-Server-IP>:3000.
	Add Prometheus as a data source.
	In Grafana, go to Settings (gear icon) ->connection -> Data Sources.
	Click Add data source.
	Select Prometheus.
	In the URL field, enter http://localhost:9090.
	Click Save & Test. You should see a green message that says "Data source is working."
 

	Import pre-built dashboards.
	Go to the Grafana dashboard and click the Dashboards (grid icon) -> Import.
	In the Import via grafana.com field, enter 1860. This is a popular dashboard for Node Exporter.
	Click Load.
	On the next page, select Prometheus as the data source and click Import.
	Repeat this process for the cAdvisor dashboard using ID 11776.
Cadvisor Dashboard
 

Node exporter dashboard
 

Step 5: Configure Log Pipeline (App → Monitoring)
	Let us update our App Server
sudo apt-get update 
	Download and install the public signing key for the Elastic repository.
sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
	Add the Elastic source list to your apt sources.
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
	Update your package list again.
sudo apt-get update
	Now, install Filebeat.
sudo apt-get install filebeat
 

We will now edit the filebeat.yml configuration file. We will configure Filebeat to collect logs from a specific path and send them to the Logstash instance on our Monitoring Server.
	Open the Filebeat configuration file on your App Server.
sudo vi /etc/filebeat/filebeat.yml
	Replace the existing content with the configuration below. This configuration will tell Filebeat to collect logs from a specific directory and output them to Logstash.
Make sure to replace <Your-Monitoring-Server-IP> with the public IP address of your Monitoring Server.
YAML
filebeat.inputs:
- type: container
  paths:
    - "/var/lib/docker/containers/*/*.log"

processors:
  - add_docker_metadata: ~
  - add_host_metadata: ~

output.logstash:
  hosts: ["<Your-Monitoring-Server-IP>:5044"]
After you have updated the file, press Esc, then type :wq and press Enter to save and quit.
	Start the Filebeat service.
sudo systemctl start filebeat
	Enable the Filebeat service to start automatically on boot.
sudo systemctl enable filebeat
	Check the status of the Filebeat service to ensure it is running without errors.
sudo systemctl status filebeat
 

The next step is to configure Logstash on your Monitoring Server to receive the logs from Filebeat and send them to Elasticsearch.
Configuring Logstash on the Monitoring Server
On your Monitoring Server, you will create a new Logstash configuration file.
	Create and open a new configuration file for Filebeat input.
sudo vi /etc/logstash/conf.d/02-beats-input.conf
	Add the following configuration to the file. This tells Logstash to listen for incoming Filebeat data on port 5044 and send it to Elasticsearch.
Note: The IP address 98.87.138.59 is your Monitoring Server's IP address and should match the one you used in your Filebeat configuration.
Code snippet
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://98.87.138.59:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
After you have added the configuration, press Esc, then type :wq and press Enter to save and quit.
Check Configuration and Restart Logstash
	Check the syntax of your Logstash configuration file.
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash/ -t
 

	If the syntax check passes, restart the Logstash service to apply the new configuration.
sudo systemctl restart logstash

Configuring Grafana for Log Visualization
Let us navigate to Grafana dashboard at http://<Your-Monitoring-Server-IP>:3000.
	Add Elasticsearch as a data source.
	In Grafana, go to Settings (gear icon) -> Data Sources.
	Click Add data source and select Elasticsearch.
	For the URL, enter http://localhost:9200.
	In the Index details section, enter filebeat-* for the Index name.
	In the Time field name field, enter @timestamp.
	Click Save & Test. You should see a green message that says "Data source is working."
 
Create a log panel in a Grafana dashboard. 
	Go to Dashboard creation
	In Grafana, click + (plus) → Dashboard → Add new panel.
	In the Query section, select Elasticsearch.
	Query type: Logs
	Lucene query
	Time range: Last 24 hours (adjust to your needs).
	Under Fields, make sure @timestamp is selected as the time field and message is in the displayed fields.
	Click Apply → name it something like "Application Logs".
 

So far, we can now collect and view the following;
	System metrics (CPU, memory, disk) – via Node Exporter
	Container metrics (running containers, resource usage) – via cAdvisor
	Application logs – via Filebeat → Logstash → Elasticsearch
	Prometheus self-metrics (Prometheus health and performance)

Step 6: Set Up Alerting
You will perform these actions on your Monitoring Server.
	Create a configuration file for Alertmanager. This file will define how alerts are routed and what receivers (e.g., email, Slack) are used.
sudo vi /etc/prometheus/alertmanager.yml
	Add the following basic configuration to the file. This sets up a default email receiver. You must replace the placeholder values with your own email and SMTP server details.
global:
  smtp_smarthost: 'smtp.example.org:587'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager@example.org'
  smtp_auth_password: 'your-email-password'

route:
  receiver: 'default-email'

receivers:
- name: 'default-email'
  email_configs:
  - to: 'your-email@example.com'
After you have added the content and replaced the placeholder values, press Esc, then type :wq and press Enter to save and quit.
	Create a systemd service file for Alertmanager.
sudo vi /etc/systemd/system/alertmanager.service

Add the following content to the service file.
[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/prometheus/alertmanager.yml --web.listen-address=0.0.0.0:9093
Restart=always

[Install]
WantedBy=multi-user.target
After you have added the content, press Esc, then type :wq and press Enter to save and quit.
	Reload the systemd daemon, enable, and start the Alertmanager service.
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
	Check the service status to make sure it is running.
sudo systemctl status alertmanager
 

Creating Prometheus Alert Rules (Continued)
You will perform these actions on your Monitoring Server.
	Create a new file for your alert rules.
sudo vi /etc/prometheus/alert.rules.yml
	Add the following content to the file. This rule will now specifically target your App Server by its instance ID.
groups:
- name: node_exporter_alerts
  rules:
  - alert: InstanceDown
    expr: up{job="node_exporter_app_server", instance="i-03362152d7d54901f:9100"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "App Server down"
      description: "The App Server (i-03362152d7d54901f) is down."

After you have added the content, press Esc, then type :wq and press Enter to save and quit.
	Now, we need to tell Prometheus to load these rules. Open your prometheus.yml file.
sudo vi /etc/prometheus/prometheus.yml
	Add the rule_files and alerting sections to the prometheus.yml file.
rule_files:
  - "alert.rules.yml"

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
After you have made these changes, press Esc, then type :wq and press Enter to save and quit.
	Restart the Prometheus service to load the new rules.
sudo systemctl restart prometheus
Check if the alert rules are active in Prometheus.
Go back to your Prometheus UI at http://<Your-Monitoring-Server-IP>:9090 and click on the Alerts tab.
You should see the InstanceDown alert rule that you created listed there. If the App Server is down, the alert's status will be FIRING.
 

Step 7: Build Master Overview Dashboard
Download & import the already created master dashboard
 
Then in Grafana:
1.	Dashboards → Import → Upload this JSON.
2.	When prompted, map datasources:
o	Prometheus → your Prometheus
o	Elasticsearch → your filebeat-* datasource (with @timestamp)
3.	Click Import.
 
 

What our Master Dashboard Shows
	Targets Health – Table showing status (UP/DOWN) of Prometheus, Node Exporter, and cAdvisor targets.
	Firing Alerts – Count of active alerts in Prometheus Alertmanager.
	CPU % (App Server) – Real-time CPU usage percentage per instance.
	Memory % Used – Real-time memory utilization percentage per instance.
	Load / Core – Load average per CPU core.
	Disk % Used (top 5 mounts) – Highest disk usage percentages for up to 5 file system mounts.
	Network In/Out (bytes/s) – Incoming and outgoing network traffic rates.
	Containers Running – Total number of running containers.
	Top-5 Containers CPU (rate) – Containers with the highest CPU usage.
	Top-5 Containers Memory (MB) – Containers with the highest memory usage (in MB).
	Log Volume (All Logs) – Time-series count of all logs ingested via Filebeat → Logstash → Elasticsearch.
	Recent Logs (All) – Live table of all logs (most recent first) with timestamp, container/host, and message.

Conclusion
The Enterprise Monitoring & Alerting Stack (EMAS) project has successfully delivered a production-grade, two-server monitoring solution capable of providing real-time metrics, container monitoring, centralized log management, and proactive alerting for distributed systems.
Starting with the deployment of dedicated Monitoring and Application servers, the project integrated a combination of industry-standard tools — Prometheus, Grafana, Alertmanager, Elasticsearch, Logstash, Filebeat, Node Exporter, and cAdvisor — to ensure a complete observability ecosystem.
From the outset, the system was designed with scalability, fault isolation, and operational visibility in mind. Metrics collection and visualization were implemented using Prometheus and Grafana, container performance insights were enabled via cAdvisor, and logs were centralized through Elasticsearch with Filebeat and Logstash pipelines. Alertmanager provided proactive notifications for threshold breaches and service issues.
The project culminated in the creation of an EMAS Master Dashboard, consolidating all critical performance indicators into one unified view — including:
	Server health status and availability
	CPU, memory, and load metrics
	Disk utilization and network throughput
	Container counts, top CPU/memory-consuming containers
	Log volume trends and detailed recent logs from Elasticsearch
	Alert states for rapid incident awareness
This final architecture not only demonstrates technical proficiency in monitoring stack deployment and integration but also provides a practical, ready-to-use production framework that can be expanded to support more servers, cloud-native workloads, and multi-cluster environments.
By completing this project, the monitoring environment is now robust, extensible, and operationally efficient, empowering DevOps teams to detect issues early, reduce downtime, and maintain optimal performance in any production-grade infrastructure.











