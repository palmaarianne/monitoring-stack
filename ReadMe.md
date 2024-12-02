Monitoring Stack Deployment with Ansible
This project automates the deployment and provisioning of a monitoring stack using Ansible and Docker Compose. It includes Grafana, Prometheus, Telegraf, and their respective configurations for datasources and dashboards.

Overview
This project demonstrates a white-label observability solution for systems and application metrics. It enables:

Automated Deployment: Deploy monitoring tools to a blank VM in minutes.
Infrastructure Automation: Leverages GitHub Actions and Ansible playbooks for consistent and reliable deployments.
Plug-and-Play Dashboards: Pre-provisioned Grafana datasources and dashboards for instant insights.

Repository Structure
monitoring-stack/
├── .github/
│   └── workflows/
│       └── deploy.yml               # GitHub Actions workflow for deployment
├── ansible.cfg                      # Ansible configuration
├── ansible-inventories/
│   ├── development/
│   │   └── inventory                # Inventory file for development
│   ├── staging/
│   │   └── inventory                # Inventory file for staging
│   └── production/
│       └── inventory                # Inventory file for production
├── ansible-playbooks/
│   └── deploy.yml                   # Ansible playbook for deployment
├── grafana-resources/
│   ├── dashboards/
│   │   └── example-dashboard.json   # Preconfigured Grafana dashboards
│   └── datasources/
│       └── datasources.yml          # Grafana datasources configuration
├── docker-compose.yml               # Docker Compose file for the monitoring stack
└── README.md                        # Documentation
Features
Automated Datasource Provisioning: Automatically configures Grafana datasources for InfluxDB and Prometheus.
Preloaded Dashboards: Grafana dashboards provisioned automatically during deployment.
GitHub Actions Integration: CI/CD pipeline for seamless deployments.
Dynamic Inventory: Supports multiple environments: development, staging, and production.
Prerequisites
Linux Host: Ensure a fresh VM with SSH access.
Ansible Installed: Version 2.16 or lower.
Docker & Docker Compose: Installed on the target host.
GitHub Repository Access: Clone this repository to start.
How to Use
Step 1: Clone the Repository
git clone https://github.com/palmaarianne/monitoring-stack.git
cd monitoring-stack
Step 2: Configure Inventory
Update the inventory file for your target environment (ansible-inventories/<environment>/inventory):

[all]
<YOUR_VM_IP> ansible_user=<SSH_USER>
Step 3: Update Variables
Modify grafana-resources/datasources/datasources.yml to match your InfluxDB and Prometheus configurations:

apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
  - name: InfluxDB
    type: influxdb
    access: proxy
    url: http://influxdb:8086
    database: telegraf
Step 4: Run Deployment via GitHub Actions
Commit your changes to your GitHub repository.
Push your branch to GitHub.
Trigger the deployment workflow from the GitHub Actions tab.
Step 5: Access Grafana
Once the deployment is complete, access Grafana at http://<YOUR_VM_IP>:3000.
Login credentials:
Username: admin
Password: admin (default, can be updated in docker-compose.yml).
Additional Notes
Supported Tools
Grafana: Visualization and alerting.
Prometheus: Metrics collection and storage.
Telegraf: System metrics collection.
Docker Compose: Orchestration of services.
Monitoring Dashboards
Place any custom Grafana dashboards under grafana-resources/dashboards/ and they will be automatically loaded during deployment.

Extending the Stack
Feel free to modify the docker-compose.yml file to add or remove services as needed.

Troubleshooting
Deployment Errors: Verify SSH access and inventory configuration.
Grafana Not Loading Dashboards: Ensure the JSON files are valid and placed in grafana-resources/dashboards/.
Datasources Not Configured: Confirm datasources.yml contains the correct URLs for Prometheus and InfluxDB.
Contributing
Contributions are welcome! Please fork the repository and submit a pull request for review.
