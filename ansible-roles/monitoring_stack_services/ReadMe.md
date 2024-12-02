# configure.monitoring_stack_services

This project uses Ansible to deploy and manage a monitoring stack on Linux hosts via Docker Compose. It provides an easy way to deploy prometheus, prometheus-exporters and telegraf

![configure-monitoring-stack-services.drawio](/uploads/e1c934fec2ada60bb9b0fb52bcb97ec8/configure-monitoring-stack-services2.jpg)

## Prerequisites

- Ansible 2.16 or lower
- SSH access to target hosts

## Features

- Dynamic service selection per host/group
- Supports first-time setup and ongoing management
- GitLab CI integration for automated deployment and management
- Allows defining prometheus scrape configuration per host/group
- Allows to configure/setup Telegraf

## Supported Exporters

- [node_exporter](https://github.com/prometheus/node_exporter)
- [cAdvisor](https://github.com/google/cadvisor/blob/master/docs/storage/prometheus.md)
- [nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter)
- [squid-exporter](https://github.com/boynux/squid-exporter)

## Supported Telegraf Plugins

- [cpu](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/cpu/README.md)
- [mem](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/mem)
- [kernel](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/kernel)
- [system](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/system)
- [processes](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/processes)
- [swap](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/swap)
- [net](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/net)
- [netstat](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/netstat)
- [disk](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/disk)
- [diskio](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/diskio)
- [conntrack](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/conntrack)
- [ntpq](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/ntpq)
- [http_response](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/http_response)
- [procstat](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/procstat)
- [ping](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/ping)
- [filestat](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/filestat)
- [x509_cert](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/x509_cert)
- [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker)
- [linux_sysctl_fs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/linux_sysctl_fs)
- [haproxy](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/haproxy)
- [redis](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/redis)
- [rabbitmq](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/rabbitmq)
- [jolokia2_agent](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/jolokia2_agent)
- [varnish](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/varnish)
- [prometheus](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/prometheus)

## Dashboard

When using Telegraf configuration the following dashboard is fully supported

- [General Linux Host Monitoring](https://grafana-lower.daeddo.com/d/qj876n2jx66qcf9/general3a-linux-host-monitoring?orgId=1)

![configure-monitoring-stack-services.drawio](/uploads/075dce3411d887207b09bec501f8ca66/image.png){width=1434 height=789}

## Usage

This project is an Ansible role, so to use it follow the above steps:

1. Create an Ansible Role folder and clone this project under it
```bash
mkdir -p ansible-roles
cd ansible-roles
git clone git@gitlab.com:sportion/shared/pipeline/deployments-ansible/ansible-roles.git
```

2. Configure ansible to use the recently created role directory
```yml
# ansible.cfg

[defaults]
host_key_checking = False
callbacks_enabled = timer, profile_tasks, profile_roles
forks=10
pipelining = True

# Path to the brand new role directory
roles_path = ./ansible-roles
```

3. Create a playbook using the `configure.monitoring_stack_services` as role
```yaml
# playbook.yml

---
- name: Configure monitoring stack services
  hosts: "{{ limit | default('all')}}"
  force_handlers: true
  gather_facts: true

  roles:
    # role configuration
    - role: 'monitoring_stack_services'
      become: true
```

## Role Variables

This project could be customized through the following variables:

### General Variables

| Variable                       | Type         | Required | Default                              | Description                                                                                                                                                                            |
|--------------------------------|--------------|----------|--------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ms_nexus_url                   | string       | no       | ""                                   | Configures Nexus private repository to install docker packages, otherwise public packages will be used.                                                                                |
| ms_config_dir                  | string       | no       | /opt/monitoring                      | folder where project configuration files will be saved.                                                                                                                                |
| ms_config_cert_dir             | string       | no       | {{ms_config_dir}}/cert               | folder where prometheus certificates will be saved.                                                                                                                                    |
| ms_compose_config_path         | string       | no       | {{ms_config_dir}}/docker-compose.yml | folder where compose configuration file will be saved.                                                                                                                                 |
| ms_docker_install_packages     | bool         | no       | true                                 | Controls if docker package should be installed.                                                                                                                                        |
| ms_services                    | list(string) | yes      | [prometheus, node_exporter]          | A list containing the names of the plugins which will be deployed. The valid values are: `Prometheus`, `node_exporter`, `nginx_exporter`, `cadvisor`, `squid-exporter` and `telegraf`. |
| ms_docker_network_ipam_subnet  | string       | no       | ""                                   | Docker custom subnet CIDR. Use when it is necessary to have control on docker private ip subnet                                                                                        |
| ms_docker_network_ipam_gateway | string       | no       | ""                                   | Docker gateway private IP on custom subnet                                                                                                                                             |                                                    |

### Prometheus Variables

| Variable                                    | Type         | Required | Default                            | Description                                                                                                                       |
|---------------------------------------------|--------------|----------|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| ms_prometheus_docker_image                  | string       | no       | prom/prometheus:latest             | Prometheus docker image used                                                                                                      |
| ms_prometheus_docker_container_name         | string       | no       | prometheus                         | Prometheus docker container name                                                                                                  |
| ms_prometheus_docker_ports                  | list(string) | no       | ['9090:9090']                      | Prometheus ports shared between host and the container                                                                            |
| ms_prometheus_docker_network_mode           | string       | no       | bridge                             | Docker networking mode used to run the container, when `bridge` the property `ms_prometheus_docker_ports` will be ignored         |
| ms_prometheus_docker_dns                    | list(string) | no       | []                                 | DNS servers used by docker prometheus container                                                                                   |
| ms_prometheus_global_labels                 | map(string)  | no       | {}                                 | Add labels (key/value pairs) on all scrapped metrics                                                                              |
| ms_prometheus_config_path                   | string       | no       | {{ ms_config_dir }}/prometheus.yml | Path where prometheus configuration will be generated                                                                             |
| ms_prometheus_scrape_interval               | string       | no       | 15s                                | Prometheus Scrape Interval                                                                                                        |
| ms_prometheus_scrape_configs_jobs           | list(object) | no       | []                                 | List of objects representing the Prometheus Scrape Jobs. More details about the configuration object are below.                   |
| ms_prometheus_remote_write_tls_enable       | bool         | no       | false                              | Configure prometheus agent to use tls protocol when sending data via remote write                                                 |
| ms_prometheus_remote_write_tls_ca_cert      | string       | no       | ""                                 | CA certificate used to authenticate to remote write url. Required when `ms_prometheus_remote_write_tls_enable` is `true`          |
| ms_prometheus_remote_write_tls_public_cert  | string       | no       | ""                                 | TLS public certificate used to authenticate to remote write url. Required when `ms_prometheus_remote_write_tls_enable` is `true`  |
| ms_prometheus_remote_write_tls_private_cert | string       | no       | ""                                 | TLS private certificate used to authenticate to remote write url. Required when `ms_prometheus_remote_write_tls_enable` is `true` |

- ### ms_prometheus_scrape_configs_job

| Variable        | Type         | Required | Default | Description                                                                            |
|-----------------|--------------|----------|---------|----------------------------------------------------------------------------------------|
| name            | string       | yes      | null    | Name of the Prometheus Scrape Job                                                      |
| targets         | list(string) | yes      | null    | List of target addresses to be scraped                                                 |
| labels          | dict         | no       | null    | Key/Value to be included in the prometheus metric                                      |
| relabel_configs | list(object) | no       | null    | List of object representing the prometheus relabel configs operations. Described below |

- ### relabel_configs

| Variable      | Type         | Required | Default | Description                                                                                    |
|---------------|--------------|----------|---------|------------------------------------------------------------------------------------------------|
| source_labels | list(string) | yes      |         | labels name which value will be selected                                                       |
| target_label  | string       | yes      |         | label name which resulting value is written in a replace action                                |
| regex         | string       | no       | (.*)    | Regular expression against which the extracted value is matched                                |
| replacement   | string       | no       | $1      | Replacement value against which a regex replace is performed if the regular expression matches |

## Node Exporter Variables

| Variable                              | Type         | Required | Default                   | Description                                                                                                                 |
|---------------------------------------|--------------|----------|---------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| ms_nodeexporter_docker_image          | string       | no       | prom/node-exporter:latest | Node Exporter docker image used                                                                                             |
| ms_nodeexporter_docker_container_name | string       | no       | node_exporter             | Node Exporter docker container name                                                                                         |
| ms_nodeexporter_docker_ports          | list(string) | no       | ['9100:9100']             | Node Exporter ports shared between host and the container                                                             |
| ms_nodeexporter_docker_network_mode   | string       | no       | bridge                    | Docker networking mode used to run the container, when `bridge` the property `ms_nodeexporter_docker_ports` will be ignored |

## Nginx Exporter Variables

| Variable                               | Type         | Required | Default                                             | Description                                                                                                                  |
|----------------------------------------|--------------|----------|-----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| ms_nginxexporter_docker_image          | string       | no       | nginx/nginx-prometheus-exporter:latest              | Nginx Exporter docker image used                                                                                             |
| ms_nginxexporter_docker_container_name | string       | no       | nginx_exporter                                      | Nginx Exporter docker container name                                                                                         |
| ms_nginxexporter_docker_ports          | list(string) | no       | ['9113:9113']                                       | Nginx Exporter ports shared between host and the container                                                                   |
| ms_nginxexporter_docker_network_mode   | string       | no       | bridge                                              | Docker networking mode used to run the container, when `bridge` the property `ms_nginxexporter_docker_ports` will be ignored |
| ms_nginxexporter_commands              | list(string) | no       | ["-nginx.scrape-uri=http://nginx:8080/stub_status"] | Arguments passed to docker container entrypoint                                                                              |

## cAdvisor Exporter Variables

| Variable                          | Type         | Required | Default                         | Description                                                                                                                 |
|-----------------------------------|--------------|----------|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| ms_cadvisor_docker_image          | string       | no       | gcr.io/cadvisor/cadvisor:latest | cAdvisor docker image used                                                                                                  |
| ms_cadvisor_docker_container_name | string       | no       | cadvisor-exporter               | cAdvisor docker container name                                                                                              |
| ms_cadvisor_docker_ports          | list(string) | no       | ['8080:8080']                   | cAdvisor ports shared between host and the container                                                                        |
| ms_cadvisor_docker_network_mode   | string       | no       | bridge                          | Docker networking mode used to run the container, when `bridge` the property `ms_nodeexporter_docker_ports` will be ignored |

## Squid Exporter Variables

| Variable                                              | Type         | Required | Default                      | Description                                                                                                                                                                                                                                     |
|-------------------------------------------------------|--------------|----------|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ms_squidexporter_docker_image                         | string       | no       | boynux/squid-exporter:latest | Squid Exporter docker image used                                                                                                                                                                                                                |
| ms_squidexporter_docker_container_name                | string       | no       | squid-exporter               | Squid Exporter docker container name                                                                                                                                                                                                            |
| ms_squidexporter_docker_ports                         | list(string) | no       | ['9301:9301']                | Squid Exporter ports shared between host and the container                                                                                                                                                                                      |
| ms_squidexporter_docker_network_mode                  | string       | no       | bridge                       | Docker networking mode used to run the container, when `bridge` the property `ms_squidexporter_docker_ports` will be ignored                                                                                                                    |
| ms_squidexporter_docker_network_ipv4_address          | string       | no       | ""                           | Private IP used by the container, it couldn't be used when `ms_squidexporter_docker_network_mode` == host, and should be a valid IP inside docker netword CIDR generated automatically by Docker or by `ms_docker_network_ipam_subnet` variable |
| ms_squidexporter_docker_env_var_squid_hostname        | string       | no       | 127.0.0.1                    | Squid's host address to be scraped                                                                                                                                                                                                              |
| ms_squidexporter_docker_env_var_squid_port            | string       | no       | 3128                         | Squid's port                                                                                                                                                                                                                                    |
| ms_squidexporter_docker_env_var_squid_exporter_listen | string       | no       | :9301                        | Squid Exporter listen port                                                                                                                                                                                                                      |

## Telegraf Variables

This section describes how to configure telegraf deployment and its variables

| Variable                              | Type         | Required | Default                                                                                                                                                                                                                                                                                   | Description                                                                                                             |
|---------------------------------------|--------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| ms_telegraf_docker_image              | string       | no       | telegraf:1.30                                                                                                                                                                                                                                                                             | Telegraf docker image used                                                                                              |
| ms_telegraf_docker_container_name     | string       | no       | telegraf                                                                                                                                                                                                                                                                                  | Telegraf docker container name                                                                                          |
| ms_telegraf_docker_privileged         | bool         | no       | false                                                                                                                                                                                                                                                                                     | Runs telegraf as priviledge user aka root, when `true` the property `ms_telegraf_docker_user` will be ignored           |
| ms_telegraf_docker_user               | string       | no       | telegraf                                                                                                                                                                                                                                                                                  | Container user used to run telegraf                                                                                     |
| ms_telegraf_docker_linux_groups       | list(string) | no       | [docker]                                                                                                                                                                                                                                                                                  | Linux groups associated with telegraf user when running the container                                                   |
| ms_telegraf_docker_network_mode       | string       | no       | host                                                                                                                                                                                                                                                                                      | Docker networking mode used to run the container, when `bridge` the property `ms_telegraf_docker_ports` will be ignored |
| ms_telegraf_docker_ports              | list(string) | no       | ['8092:8092/udp', '8094:8094/tcp', '8125:8125/udp']                                                                                                                                                                                                                                       | Telegraf ports shared between host and the container                                                                    |
| ms_telegraf_docker_socket_path        | string       | no       | /var/run/docker.sock                                                                                                                                                                                                                                                                      | Docker socket path on host, used to monitor docker                                                                      |
| ms_telegraf_config_path               | string       | no       | {{monitoring_stack_config_dir}}/telegraf.conf"                                                                                                                                                                                                                                            | Path where telegraf configuration will be generated                                                                     |
| ms_telegraf_global_tags               | map(string)  | no       | {}                                                                                                                                                                                                                                                                                        | Tags to be included in each telegraf metrics. Ex { environment: "perf" }                                                |
| ms_telegraf_agent_interval            | string       | no       | 10s                                                                                                                                                                                                                                                                                       | Default data collection interval for all telegraf inputs.                                                               |
| ms_telegraf_agent_flush_interval      | string       | no       | 10s                                                                                                                                                                                                                                                                                       | Default flushing interval for all outputs.                                                                              |
| ms_telegraf_agent_hostname            | string       | no       | {{ ansible_hostname }}                                                                                                                                                                                                                                                                    | Value of host tag on telegraf metrics                                                                                   |
| ms_telegraf_input_cpu                 | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Enables telegraf cpu input plugin                                                                                       |
| ms_telegraf_input_mem                 | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Enables telegraf mem input plugin                                                                                       |
| ms_telegraf_input_kernel              | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf kernel input plugin                                                                                  |
| ms_telegraf_input_system              | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf system input plugin                                                                                  |
| ms_telegraf_input_processes           | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf processes input plugin                                                                               |
| ms_telegraf_input_swap                | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf swap input plugin                                                                                    |
| ms_telegraf_input_net                 | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf net input plugin                                                                                     |
| ms_telegraf_input_netstat             | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf netstat input plugin                                                                                 |
| ms_telegraf_input_disk                | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf disk input plugin                                                                                    |
| ms_telegraf_input_diskio              | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf diskio input plugin                                                                                  |
| ms_telegraf_input_conntrack           | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf conntrack input plugin                                                                               |
| ms_telegraf_input_ntpq                | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf ntpq input plugin                                                                                    |
| ms_telegraf_input_http_response       | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf http_response input plugin. More details about the configuration object are below.                   |
| ms_telegraf_input_procstat            | object       | no       | {    enabled: false,   monitored_system_processes: [{ name: "telegraf"}] }                                                                                                                                                                                                                | Configure telegraf procstat input plugin. More details about the configuration object are below.                        |
| ms_telegraf_input_ping                | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf ping input plugin. More details about the configuration object are below.                            |
| ms_telegraf_input_filestat            | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf filestat input plugin. More details about the configuration object are below.                        |
| ms_telegraf_input_x509_cert           | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf x509 input plugin. More details about the configuration object are below.                            |
| ms_telegraf_input_docker              | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf docker input plugin. More details about the configuration object are below.                          |
| ms_telegraf_input_linux_sysctl_fs     | object       | no       | { enabled: true }                                                                                                                                                                                                                                                                         | Configure telegraf linux_sysctl_fs input plugin.                                                                        |
| ms_telegraf_input_haproxy             | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf haproxy input plugin. More details about the configuration object are below.                         |
| ms_telegraf_input_redis               | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf redis input plugin. More details about the configuration object are below.                           |
| ms_telegraf_input_rabbitmq            | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf rabbitmq input plugin. More details about the configuration object are below.                        |
| ms_telegraf_input_prometheus          | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf prometheus input plugin. More details about the configuration object are below.                      |
| ms_telegraf_input_jolokia2_agent      | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf jolokia2_agent input plugin. More details about the configuration object are below.                  |
| ms_telegraf_input_varnish             | object       | no       | { enabled: false }                                                                                                                                                                                                                                                                        | Configure telegraf varnish input plugin. More details about the configuration object are below.                         |
| ms_telegraf_input_varnish_folder_path | object       | no       | /var/lib/varnish                                                                                                                                                                                                                                                                          | Varnish directory to extract metrics                                                                                    |
| ms_telegraf_input_varnish_libraries   | list(string) | no       | [   /usr/lib64/libvarnishapi.so.2:/usr/lib/x86_64-linux-gnu/libvarnishapi.so.2   /usr/lib64/libncursesw.so.5:/usr/lib/x86_64-linux-gnu/libncursesw.so.5   /usr/lib64/libtinfo.so.5:/usr/lib/x86_64-linux-gnu/libtinfo.so.5   /usr/lib64/libpcre.so.1:/lib/x86_64-linux-gnu/libpcre.so.1 ] | Varnish libraries required to varnishstat to work. Obtained by running `ldd varnishstat`.                               |
| ms_telegraf_output_influxdb           | list(object) | no       | []                                                                                                                                                                                                                                                                                        | Configure telegraf influxdbv1 output plugin. More detail about the configuration object are below.                      |
| ms_telegraf_output_kafka              | list(object) | no       | []                                                                                                                                                                                                                                                                                        | Configure telegraf kafka output plugin. More detail about the configuration object are below.                           |                         |

### Telegraf Input Plugins

Every single telegraf input plugin object (ms_telegraf_input_cpu, ms_telegraf_input_prometheus, ms_telegraf_input_* ) has these variables

- ### ms_telegraf_input_*

| Variable | Type        | Required | Default | Description                                                                                |
|----------|-------------|----------|---------|--------------------------------------------------------------------------------------------|
| enabled  | bool        | no       | true    | Activate/deactivate the plugin                                                             |
| tags     | map(string) | no       | {}      | Tags to be included in the plugin metric. Ex { tier: "memory-intensive", owner: "team-A" } |

- ### ms_telegraf_input_http_response

| Variable                     | Type         | Required | Default | Description                                                                                                                     |
|------------------------------|--------------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------|
| application_healthcheck_urls | list(object) | no       | []      | List of application healthcheck objects to be monitored. More details on `application_healthcheck_url` object definition below  |

- ### application_healthcheck_url

| Variable              | Type         | Required | Default | Description                                                                |
|-----------------------|--------------|----------|---------|----------------------------------------------------------------------------|
| urls                  | list(string) | yes      | -       | List of urls to be monitored. Ex [ "http://localhost:8888" ]               |
| method                | string       | no       | GET     | HTTP method used                                                           |
| interval              | string       | no       | 15s     | Default data collection interval                                           |
| response_status_code  | number       | no       | 200     | Expected response status code                                              |
| response_string_match | string       | no       | -       | Optional substring or regex match in body of the response (case sensitive) |

- ### ms_telegraf_input_procstat

| Variable                     | Type         | Required | Default | Description                                                                                                                     |
|------------------------------|--------------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------|
| monitored_system_processes | list(object) | no       | []      | List of monitored_system_processes objects to be monitored. More details on `monitored_system_process` object definition below  |

- ### monitored_system_process

| Variable | Type   | Required | Default | Description                                                                                                                                                   |
|----------|--------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name     | string | yes      | -       | Name of the process monitored, this value will be used to label the process name on the metric extracted                                                      |
| interval | string | no       | 10s     | Default data collection interval                                                                                                                              |
| pattern  | string | no       | -       | If defined this pattern will be used on `pgrep` to find out process information, if it is not defined the search will be performed using the `name` attribute |

- ### ms_telegraf_input_ping

| Variable                     | Type         | Required | Default | Description                                                                                                                     |
|------------------------------|--------------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------|
| monitored_ping_urls | list(object) | no       | []      | List of monitored_ping_urls objects to be monitored. More details on `monitored_ping_url` object definition below  |

- ### monitored_ping_url

| Variable | Type         | Required | Default | Description                                      |
|----------|--------------|----------|---------|--------------------------------------------------|
| urls     | list(string) | yes      | -       | Hosts to send ping packets to                    |
| count    | number       | no       | 5       | Number of ping packets to send per interval.     |
| timeout  | number       | no       | 2       | the time to wait for a ping response in seconds. |


- ### ms_telegraf_input_filestat

| Variable | Type         | Required | Default | Description                                                                          |
|----------|--------------|----------|---------|--------------------------------------------------------------------------------------|
| files    | list(string) | yes      | -       | Files to gather stats about. Ex. [["/etc/telegraf/telegraf.conf", "/var/log/**.log"] |

- ### ms_telegraf_input_x509_cert

| Variable           | Type         | Required | Default | Description                                                                                                                             |
|--------------------|--------------|----------|---------|-----------------------------------------------------------------------------------------------------------------------------------------|
| sources            | list(string) | yes      | -       | List certificate sources. Ex: ["https://influxdata.com:443", "/etc/ssl/certs/ssl-cert-snakeoil.pem", "/etc/mycerts/*.mydomain.org.pem"] |
| interval           | string       | no       | 60s     | Default data collection interval                                                                                                        |
| exclude_root_certs | bool         | no       | true    | Only output the leaf certificates and omit the root ones.                                                                               |

- ### ms_telegraf_input_docker

| Variable                | Type         | Required | Default                     | Description                                                                                                                                                                                                   |
|-------------------------|--------------|----------|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| endpoint                | string       | no       | unix:///var/run/docker.sock | Docker Endpoint                                                                                                                                                                                               |
| container_name_include  | list(string) | no       | []                          | Containers to include. Collect all if empty. Globs accepted.                                                                                                                                                  |
| container_name_exclude  | list(string) | no       | []                          | Containers to exclude.                                                                                                                                                                                        |
| container_state_include | list(string) | no       | []                          | Container states to include. Globs accepted. When empty only containers in the "running" state will be captured. Valid values are: "created", "restarting", "running", "removing", "paused", "exited", "dead" |
| container_state_exclude | list(string) | no       | []                          | Container states to exclude. Globs accepted. Valid values are: "created", "restarting", "running", "removing", "paused", "exited", "dead"                                                                     |

- ### ms_telegraf_input_haproxy

| Variable | Type         | Required | Default | Description                                                                                                                                                      |
|----------|--------------|----------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| servers  | list(string) | true     | -       | List of stats endpoints. Metrics can be collected from both http and socket. Ex: [ "socket:/run/haproxy/admin.sock", "http://myhaproxy.com:1936/haproxy?stats" ] |

- ### ms_telegraf_input_redis

| Variable | Type         | Required | Default | Description                                                                                                                     |
|----------|--------------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------|
| servers  | list(string) | true     | -       | List of Redis endpoints. Ex: [ "tcp://localhost:6379", "unix:///var/run/redis.sock", "tcp://username:password@192.168.99.100" ] |

- ### ms_telegraf_input_rabbitmq

| Variable       | Type         | Required | Default | Description                                                                                     |
|----------------|--------------|----------|---------|-------------------------------------------------------------------------------------------------|
| url            | string       | true     | -       | Management Plugin url. Ex: http://localhost:1567                                                |
| username       | string       | true     | -       | Username Credential                                                                             |
| password       | string       | true     | -       | Password Credential                                                                             |
| metric_exclude | list(string) | no       | []      | Metrics to be excluded. Valid values are: "exchange", "federation", "node", "overview", "queue" |

- ### jolokia2_agent_config

| Variable    | Type         | Required | Default | Description                                                                                                                    |
|-------------|--------------|----------|---------|--------------------------------------------------------------------------------------------------------------------------------|
| urls        | list(string) | true     | -       | agents URLs to query                                                                                                           |
| name_prefix | string       | no       | ""      | Prefix to be added on measurement name                                                                                         |
| metrics     | list(string) | true     | -       | list of jolokia_metric object to be queried using the provided URLs. More details on  `jolokia_metric` object definition below |

- ### jolokia_metric

| Variable     | Type         | Required | Default | Description                                                                                                                                              |
|--------------|--------------|----------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| name         | string       | true     | -       | Measurement name                                                                                                                                         |
| mbean        | string       | true     | -       | The object name of a JMX MBean. MBean property-key values can contain a wildcard *, allowing you to fetch multiple MBeans with one declaration.          |
| paths        | list(string) | false    | -       | A list of MBean attributes to read.                                                                                                                      |
| tag_keys     | list(string) | false    | -       | A list of MBean property-key names to convert into tags. The property-key name becomes the tag name, while the property-key value becomes the tag value. |
| field_prefix | string       | false    | -       | A string to prepend to the tag names produced by this metric declaration.                                                                                |
| field_name   | string       | false    | -       | A string to set as the name of the field produced by this metric; can contain substitutions.                                                             |

- ### ms_telegraf_input_varnish

| Variable       | Type         | Required | Default | Description                                                                                     |
|----------------|--------------|----------|---------|-------------------------------------------------------------------------------------------------|
| binary         | string       | true     | -       | The default location of the varnishstat binary                                                  |
| adm_binary     | string       | true     | -       | The default location of the varnishadm binary                                                   |
| metric_version | number       | false    | 1       | Metric version defaults to metric_version=1, use metric_version=2 for removal of nonactive vcls |
| stats          | list(string) | false    | ['*']   | Glob matching can be used, ie, stats = ["MAIN.*"]                                               |

- ### ms_telegraf_input_prometheus

| Variable       | Type         | Required | Default | Description                                                                       |
|----------------|--------------|----------|---------|-----------------------------------------------------------------------------------|
| urls           | list(string) | true     | -       | urls to scrape metrics from                                                       |
| metric_version | string       | false    | 1       | Metric version controls the mapping from Prometheus metrics into Telegraf metrics |

### Telegraf Output Plugins

Every single telegraf output plugin object (ms_telegraf_output_kafka, ms_telegraf_output_influxdb ) has these variables

- ### ms_telegraf_output_*

| Variable   | Type         | Required | Default | Description                                                                                                                                                             |
|------------|--------------|----------|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| enabled    | bool         | no       | true    | Activate/deactivate the plugin                                                                                                                                          |
| tagexclude | list(string) | no       | []      | Tags with a tag key matching one of the patterns will be discarded from the metric                                                                                      |
| tagdrop    | map(string)  | no       | {}      | The inverse of tagpass. If a match is found the metric is discarded.                                                                                                    |
| tagpass    | map(string)  | no       | {}      | A table mapping tag keys to arrays of glob pattern strings. Only metrics that contain a tag key in the table and a tag value matching one of its patterns is processed. |

- ### ms_telegraf_output_influxdb

This variable is a list of the objects defined below

| Variable               | Type         | Required | Default  | Description                                                                        |
|------------------------|--------------|----------|----------|------------------------------------------------------------------------------------|
| urls                   | list(string) | true     | -        | The full HTTP or UDP URL for your InfluxDB instance. Ex: ["http://127.0.0.1:8086"] |
| database               | string       | false    | telegraf | The target database for metrics                                                    |
| skip_database_creation | bool         | false    | false    | If true, no CREATE DATABASE queries will be sent.                                  |

- ### ms_telegraf_output_kafka

This variable is a list of the objects defined below

| Variable          | Type         | Required | Default  | Description                                                                                                                                                                    |
|-------------------|--------------|----------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| brokers           | list(string) | true     | -        | URLs of kafka brokers. Ex: ["localhost:9092"]                                                                                                                                  |
| topic             | string       | false    | telegraf | Kafka topic for producer messages                                                                                                                                              |
| routing_tag       | string       | false    | host     | The routing tag specifies a tagkey on the metric whose value is used as the message key                                                                                        |
| required_acks     | number       | false    | 0        | used in Produce Requests to tell the broker how many replica acknowledgements it must see before responding. 0 -> No wait; 1 -> ack for leader; -1 -> ack for in-sync replicas |
| max_retry         | number       | false    | 3        | The maximum number of times to retry sending a metric before failing until the next flush                                                                                      |
| compression_codec | numbber      | false    | 0        | Compression codec represents the various compression codecs recognized by kafka. 0 -> None; 1 -> Gzip; 2 -> Snappy; 3 -> LZ4; 4 -> ZSTD                                        |

### Telegraf Minimal Configuration
The configuration provided below enables the default plugins such as: cpu, mem, kernel, docker and etc.
```yaml
monitoring_stack_services:
  - telegraf

ms_telegraf_output_influxdb:
  - urls: [ "http://dsblower-influxdb17.private.daeddo.com:8086" ]
    database: "telegraf"
```

### Telegraf General Configuration
The configuration provided below enables the default plugins and fits in most of the use case scenarios, monitoring processes, endpoints and connectivity
```yaml
monitoring_stack_services:
  - telegraf

ms_telegraf_input_procstat:
  monitored_system_processes:
    - name: filebeat
    - name: telegraf
    - name: backend-api

ms_telegraf_input_http_response:
  application_healthcheck_urls:
    - urls:
      - "http://localhost:3010"
      - "http://localhost:8080/abp/health"

ms_telegraf_input_ping:
  monitored_ping_urls:
    - urls: ["oeuf1dc-dsbl1-mess01.development.cloud"]

ms_telegraf_output_influxdb:
  - urls: [ "http://dsblower-influxdb17.private.daeddo.com:8086" ]
    database: "telegraf"
```
