<p align="center">
  <img src="https://github.com/veteranbv/may8-chatbotshowdown/raw/main/immagptchatbotvs.jpg" alt="Screenshot of the chat vs">
</p>

# Ansible Playbook Challenge: "I'm a good gpt2" vs "I'm also a good gpt2" Showdown

I challenged two LLM models, "im-a-good-gpt2-chatbot" and "im-also-a-good-gpt2-chatbot", on the [chat.lmsys.org](https://chat.lmsys.org/) platform to create an Ansible playbook for a fun homelab project. The models were provided with an example of my existing Ansible directory structure to guide their responses:

```
.
├── ansible.cfg
├── inventory
│   ├── group_vars
│   │   ├── all.yml
│   │   └── raspberrypi.yml
│   ├── production
│   │   └── hosts
│   └── staging
│       └── hosts
├── playbooks
│   └── raspi_cleanup
│       └── cleanup_restore_raspi_to_default.yaml
├── requirements.yml
├── roles
│   └── raspi_cleanup
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       ├── meta
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       └── vars
│           └── main.yml
└── scripts
    ├── create_ansible_project.sh
    └── create_inventory.sh
```

## Analysis

Both ChatGPT models provided impressive and thorough responses to the challenge, demonstrating their ability to generate creative and structured Ansible playbooks.

Model A, "im-a-good-gpt2-chatbot", created a comprehensive media server setup using Docker, Docker Compose, and popular applications like Plex, Sonarr, Radarr, and Transmission. The response was well-organized, adhered to Ansible best practices, and included a detailed playbook, role, inventory adjustments, and execution instructions.

Model B, "im-also-a-good-gpt2-chatbot", designed a Grafana monitoring stack integrated with Prometheus for system metrics collection. While also well-structured, Model B's response didn't quite match the level of detail and organization provided by Model A. However, the inclusion of the node_exporter role was a nice touch.

Overall, both models demonstrated strong capabilities in generating Ansible playbooks, with Model A slightly edging out Model B in terms of structure and thoroughness.

## Prompt

> Help me write an ansible playbook and accompanying files for a fun homelab project of your choice, but ensure that it's done thoroughly leveraging the structure of my current ansible directory so you can get an idea of how I want to structure new projects: [tree pasted here]

## Initial Comparisons

### Model A: im-a-good-gpt2-chatbot

<details>
  <summary>Click to expand Model A's response</summary>

~~~
Given your existing Ansible structure and preferences, let's create a fun and practical homelab project. We'll set up a **Media Server** using **Docker**, **Docker Compose**, and **Portainer** for easy container management. This setup will help you host and manage various media applications like Plex, Sonarr, Radarr, and Transmission in a structured and maintainable way.

Here’s how we’ll structure this project within your existing Ansible setup:

### Project Overview

- **Project Name**: `media_server_setup`
- **Key Components**:
  - **Docker**: To run containers.
  - **Docker Compose**: To manage multi-container Docker applications.
  - **Portainer**: To provide a web-based UI for Docker.
  - **Plex**: For media playback.
  - **Sonarr**: For TV series management.
  - **Radarr**: For movies management.
  - **Transmission**: For downloading torrents.

### Directory Structure

We will create the following structure under the `playbooks` and `roles` directories:

```
playbooks/
└── media_server_setup
    └── deploy_media_server.yml

roles/
└── media_server_setup
    ├── defaults
    │   └── main.yml
    ├── files
    │   └── docker-compose.yml
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    └── vars
        └── main.yml
```

### Step-by-Step Implementation

#### 1. Define the Playbook

**File**: `playbooks/media_server_setup/deploy_media_server.yml`

```yaml
---
- name: Deploy Media Server
  hosts: media_servers
  become: yes
  roles:
    - media_server_setup
```

#### 2. Role Variables

**File**: `roles/media_server_setup/defaults/main.yml`

```yaml
---
# Docker and application versions
docker_compose_version: "1.29.2"
plex_version: "latest"
sonarr_version: "latest"
radarr_version: "latest"
transmission_version: "latest"

# Configuration directories
media_server_directory: "/opt/media_server"
docker_compose_file: "{{ media_server_directory }}/docker-compose.yml"
```

**File**: `roles/media_server_setup/vars/main.yml`

```yaml
---
# This file can hold any variables specific to the media server setup that you want to override from defaults.
```

#### 3. Main Task to Set Up the Media Server

**File**: `roles/media_server_setup/tasks/main.yml`

```yaml
---
- name: Ensure Docker is installed
  ansible.builtin.package:
    name: docker
    state: present

- name: Install Docker Compose
  ansible.builtin.get_url:
    url: "https://github.com/docker/compose/releases/download/{{ docker_compose_version }}/docker-compose-{{ ansible_system }}-{{ ansible_architecture }}"
    dest: "/usr/local/bin/docker-compose"
    mode: '0755'

- name: Ensure the media server directory exists
  ansible.builtin.file:
    path: "{{ media_server_directory }}"
    state: directory
    mode: '0755'

- name: Deploy Docker Compose file
  ansible.builtin.template:
    src: docker-compose.yml
    dest: "{{ docker_compose_file }}"

- name: Pull and start up all containers
  ansible.builtin.command:
    cmd: docker-compose -f "{{ docker_compose_file }}" up -d
  args:
    chdir: "{{ media_server_directory }}"
```

#### 4. Handlers to Manage Services

**File**: `roles/media_server_setup/handlers/main.yml`

```yaml
---
- name: restart docker
  ansible.builtin.service:
    name: docker
    state: restarted
```

#### 5. Docker Compose Template

**File**: `roles/media_server_setup/files/docker-compose.yml`

```yaml
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

  plex:
    image: plexinc/pms-docker:{{ plex_version }}
    container_name: plex
    restart: unless-stopped
    ports:
      - "32400:32400"
    volumes:
      - ./data/plex:/config
      - ./media:/media

  sonarr:
    image: linuxserver/sonarr:{{ sonarr_version }}
    container_name: sonarr
    restart: unless-stopped
    ports:
      - "8989:8989"
    volumes:
      - ./data/sonarr:/config
      - ./media/tv:/tv
      - ./downloads:/downloads

  radarr:
    image: linuxserver/radarr:{{ radarr_version }}
    container_name: radarr
    restart: unless-stopped
    ports:
      - "7878:7878"
    volumes:
      - ./data/radarr:/config
      - ./media/movies:/movies
      - ./downloads:/downloads

  transmission:
    image: linuxserver/transmission:{{ transmission_version }}
    container_name: transmission
    restart: unless-stopped
    ports:
      - "9091:9091"
      - "51413:51413"
      - "51413:51413/udp"
    volumes:
      - ./data/transmission:/config
      - ./downloads:/downloads

volumes:
  portainer_data:
```

### Inventory Adjustments

Make sure your `inventory/production/hosts` or `inventory/staging/hosts` files have a group `[media_servers]` with the IPs or hostnames of the servers where you want to deploy this setup.

```ini
[media_servers]
192.168.1.100
192.168.1.101
```

### Execution

To execute this playbook, navigate to your Ansible directory and run:

```bash
ansible-playbook playbooks/media_server_setup/deploy_media_server.yml -i inventory/production/hosts
```

### Conclusion

This setup gives you a robust starting point for a media server in your homelab. You can extend the `docker-compose.yml` with more services as needed, and update your roles and playbooks to customize further. This project structure and approach keep your configurations maintainable and clear.
~~~

</details>

### Model B: im-also-a-good-gpt2-chatbot

<details>
  <summary>Click to expand Model B's response</summary>
  
~~~
Here's a detailed Ansible playbook and accompanying files for a fun homelab project. For this, let's set up a **Grafana** monitoring system on a dedicated server. Grafana is great for visualizing data from various sources like Prometheus, InfluxDB, etc. We'll also include Prometheus to gather system metrics.

### Directory Structure - Update
```
.
├── ansible.cfg
├── inventory
│   ├── group_vars
│   │   ├── all.yml
│   │   ├── grafana.yml
│   │   └── prometheus.yml
│   ├── production
│   │   └── hosts
│   └── staging
│       └── hosts
├── playbooks
│   └── monitoring_stack
│       └── deploy_monitoring_stack.yml
└── roles
    ├── grafana
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   └── grafana.ini.j2
    │   └── vars
    │       └── main.yml
    ├── prometheus
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   │   └── prometheus.yml.j2
    │   └── vars
    │       └── main.yml
    └── node_exporter
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── node_exporter.service.j2
```

### Inventory Files

#### `inventory/production/hosts`

```ini
[monitoring]
monitoring-node-1 ansible_host=<YOUR_SERVER_IP> ansible_user=<YOUR_USER>
```

#### `inventory/group_vars/all.yml`

```yaml
---
ansible_python_interpreter: /usr/bin/python3
```

#### `inventory/group_vars/grafana.yml`

```yaml
---
grafana_version: "9.2.5"
grafana_admin_password: "admin"
```

#### `inventory/group_vars/prometheus.yml`

```yaml
---
prometheus_version: "2.41.0"
node_exporter_version: "1.5.0"
```

### Playbook

#### `playbooks/monitoring_stack/deploy_monitoring_stack.yml`

```yaml
---
- name: Deploy Monitoring Stack
  hosts: monitoring
  become: true
  roles:
    - prometheus
    - grafana
    - node_exporter
```

### Roles

#### Prometheus Role

##### `roles/prometheus/defaults/main.yml`

```yaml
---
prometheus_version: "2.41.0"
node_exporter_version: "1.5.0"
```

##### `roles/prometheus/handlers/main.yml`

```yaml
---
- name: Restart Prometheus
  service:
    name: prometheus
    state: restarted
```

##### `roles/prometheus/meta/main.yml`

```yaml
---
dependencies: []
```

##### `roles/prometheus/tasks/main.yml`

```yaml
---
- name: Download Prometheus
  get_url:
    url: "https://github.com/prometheus/prometheus/releases/download/v{{ prometheus_version }}/prometheus-{{ prometheus_version }}.linux-amd64.tar.gz"
    dest: "/tmp/prometheus.tar.gz"

- name: Extract Prometheus
  unarchive:
    src: "/tmp/prometheus.tar.gz"
    dest: "/usr/local/"
    remote_src: true
  notify: Restart Prometheus

- name: Create Prometheus user
  user:
    name: prometheus
    shell: /usr/sbin/nologin

- name: Create directories
  file:
    path: "{{ item }}"
    state: directory
    owner: prometheus
    group: prometheus
  with_items:
    - /etc/prometheus
    - /var/lib/prometheus

- name: Copy Prometheus binary
  copy:
    src: "/usr/local/prometheus-{{ prometheus_version }}.linux-amd64/prometheus"
    dest: "/usr/local/bin/prometheus"
    owner: prometheus
    group: prometheus
    mode: "0755"

- name: Copy Prometheus config
  template:
    src: prometheus.yml.j2
    dest: /etc/prometheus/prometheus.yml
  notify: Restart Prometheus

- name: Create systemd service
  copy:
    content: |
      [Unit]
      Description=Prometheus
      Wants=network-online.target
      After=network-online.target

      [Service]
      User=prometheus
      Group=prometheus
      Type=simple
      ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus

      [Install]
      WantedBy=multi-user.target
    dest: /etc/systemd/system/prometheus.service
  notify: Restart Prometheus

- name: Enable and start Prometheus
  systemd:
    name: prometheus
    enabled: true
    state: started
```

##### `roles/prometheus/templates/prometheus.yml.j2`

```yaml
---
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

##### `roles/prometheus/vars/main.yml`

```yaml
---
prometheus_version: "2.41.0"
```

#### Grafana Role

##### `roles/grafana/defaults/main.yml`

```yaml
---
grafana_version: "9.2.5"
grafana_admin_password: "admin"
```

##### `roles/grafana/handlers/main.yml`

```yaml
---
- name: Restart Grafana
  service:
    name: grafana-server
    state: restarted
```

##### `roles/grafana/meta/main.yml`

```yaml
---
dependencies: []
```

##### `roles/grafana/tasks/main.yml`

```yaml
---
- name: Add Grafana repository
  apt_repository:
    repo: "deb https://packages.grafana.com/oss/deb stable main"
    state: present
    filename: grafana

- name: Add Grafana GPG key
  apt_key:
    url: "https://packages.grafana.com/gpg.key"
    state: present

- name: Update APT cache
  apt:
    update_cache: yes

- name: Install Grafana
  apt:
    name: grafana
    state: present
  notify: Restart Grafana

- name: Configure Grafana
  template:
    src: grafana.ini.j2
    dest: /etc/grafana/grafana.ini
  notify: Restart Grafana

- name: Enable and start Grafana
  systemd:
    name: grafana-server
    enabled: yes
    state: started
```

##### `roles/grafana/templates/grafana.ini.j2`

```ini
[security]
admin_user = admin
admin_password = {{ grafana_admin_password }}
```

##### `roles/grafana/vars/main.yml`

```yaml
---
grafana_version: "9.2.5"
```

#### Node Exporter Role

##### `roles/node_exporter/defaults/main.yml`

```yaml
---
node_exporter_version: "1.5.0"
```

##### `roles/node_exporter/handlers/main.yml`

```yaml
---
- name: Restart Node Exporter
  systemd:
    name: node_exporter
    state: restarted
```

##### `roles/node_exporter/meta/main.yml`

```yaml
---
dependencies: []
```

##### `roles/node_exporter/tasks/main.yml`

```yaml
---
- name: Download Node Exporter
  get_url:
    url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
    dest: "/tmp/node_exporter.tar.gz"

- name: Extract Node Exporter
  unarchive:
    src: "/tmp/node_exporter.tar.gz"
    dest: "/usr/local/"
    remote_src: true
  notify: Restart Node Exporter

- name: Create Node Exporter user
  user:
    name: node_exporter
    shell: /usr/sbin/nologin

- name: Copy Node Exporter binary
  copy:
    src: "/usr/local/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter"
    dest:
~~~

</details>

## Additional Comparisons

While my initial interest in running this test was to assess the exiting new gpt2 models that were in the wild, I thought it might be interesting to also run the same prompt on the other major models right now. In that effort, I've included the results of running the prompt on ChatGPT - GPT-4, Claude 3 Opus, Google AI Studio - Gemini 1.5 Pro, Llama 3, Llama 3:70B, and Mistral Large. 

### Model C: ChatGPT - GPT-4

<details>
  <summary>Click to expand Model C's response</summary>
  
~~~
Let's create an Ansible project for setting up a Minecraft server as a fun homelab project. This will be a thorough implementation that includes the structure, roles, and playbook files for deploying a Minecraft server on a Linux host. This project will demonstrate how to use Ansible for configuration management and deployment tasks. 

### Project Structure

The new project will be structured as follows, fitting into your existing Ansible directory setup:

```
playbooks/
└── minecraft_server_setup
    └── deploy_minecraft.yml

roles/
└── minecraft_server
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    └── vars
        └── main.yml
```

### Step 1: Role Defaults

**roles/minecraft_server/defaults/main.yml**

```yaml
---
minecraft_user: minecraft
minecraft_group: minecraft
minecraft_version: "1.16.5"
minecraft_server_dir: "/opt/minecraft"
minecraft_xms: "1024M"
minecraft_xmx: "2048M"
```

### Step 2: Handlers

**roles/minecraft_server/handlers/main.yml**

```yaml
---
- name: restart minecraft
  systemd:
    name: minecraft
    state: restarted
  become: true
```

### Step 3: Tasks

**roles/minecraft_server/tasks/main.yml**

```yaml
---
- name: Ensure group exists
  group:
    name: "{{ minecraft_group }}"
    state: present

- name: Ensure user exists
  user:
    name: "{{ minecraft_user }}"
    group: "{{ minecraft_group }}"
    home: "{{ minecraft_server_dir }}"
    createhome: no

- name: Install required packages
  apt:
    name: openjdk-11-jre-headless
    state: present
  become: true

- name: Create Minecraft server directory
  file:
    path: "{{ minecraft_server_dir }}"
    state: directory
    owner: "{{ minecraft_user }}"
    group: "{{ minecraft_group }}"

- name: Download Minecraft Server
  get_url:
    url: "https://launcher.mojang.com/v1/objects/f1a0073671057f01aa843443fef34330281333ce/server.jar"
    dest: "{{ minecraft_server_dir }}/server.jar"
    mode: '0755'
  when: minecraft_version is defined

- name: Create systemd service file
  template:
    src: minecraft.service.j2
    dest: "/etc/systemd/system/minecraft.service"
  notify: restart minecraft
  become: true

- name: Enable and start Minecraft service
  systemd:
    name: minecraft
    enabled: yes
    state: started
  become: true
```

### Step 4: Templates

**roles/minecraft_server/templates/minecraft.service.j2**

```jinja
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User={{ minecraft_user }}
Nice=1
KillMode=none
SuccessExitStatus=0 1
ProtectHome=true
ProtectSystem=full
PrivateDevices=true
NoNewPrivileges=true
ExecStart=/usr/bin/java -Xms{{ minecraft_xms }} -Xmx{{ minecraft_xmx }} -jar {{ minecraft_server_dir }}/server.jar nogui
WorkingDirectory={{ minecraft_server_dir }}

[Install]
WantedBy=multi-user.target
```

### Step 5: Playbook

**playbooks/minecraft_server_setup/deploy_minecraft.yml**

```yaml
---
- name: Deploy Minecraft Server
  hosts: all
  become: true
  roles:
    - minecraft_server
```

### Conclusion

You can execute this playbook using your existing inventory and configuration by navigating to your Ansible directory and running:

```bash
ansible-playbook playbooks/minecraft_server_setup/deploy_minecraft.yml -i inventory/production/hosts
```

This setup installs a Minecraft server using OpenJDK 11 and configures it to run as a service on your system. Adjust the memory settings and Minecraft version according to your server's capacity and the version you prefer.
~~~

</details>

### Model D: Anthropic - Claude 3 Opus

<details>
  <summary>Click to expand Model D's response</summary>
  
~~~
Great! Let's create an Ansible playbook for a fun homelab project - setting up a Minecraft server on a Raspberry Pi.

First, let's create a new role called `minecraft_server`:

```bash
mkdir -p roles/minecraft_server/{defaults,files,handlers,meta,tasks,templates,vars}
touch roles/minecraft_server/{defaults,vars}/main.yml
touch roles/minecraft_server/meta/main.yml
touch roles/minecraft_server/tasks/main.yml
touch roles/minecraft_server/handlers/main.yml
```

Now, let's populate the role files:

`roles/minecraft_server/defaults/main.yml`:
```yaml
---
minecraft_version: 1.18.2
minecraft_server_path: /opt/minecraft
minecraft_server_port: 25565
minecraft_max_memory: 1024M
minecraft_min_memory: 512M
```

`roles/minecraft_server/vars/main.yml`:
```yaml
---
minecraft_url: https://launcher.mojang.com/v1/objects/c8f83c5655308435b3dcf03c06d9fe8740a77469/server.jar
```

`roles/minecraft_server/tasks/main.yml`:
```yaml
---
- name: Install required packages
  apt:
    name:
      - openjdk-17-jre-headless
      - screen
    state: present

- name: Create Minecraft server directory
  file:
    path: "{{ minecraft_server_path }}"
    state: directory

- name: Download Minecraft server
  get_url:
    url: "{{ minecraft_url }}"
    dest: "{{ minecraft_server_path }}/server.jar"

- name: Accept Minecraft EULA
  copy:
    content: "eula=true"
    dest: "{{ minecraft_server_path }}/eula.txt"

- name: Configure Minecraft server
  template:
    src: server.properties.j2
    dest: "{{ minecraft_server_path }}/server.properties"
  notify: restart minecraft

- name: Create Minecraft systemd service
  template:
    src: minecraft.service.j2
    dest: /etc/systemd/system/minecraft.service
  notify: restart minecraft
```

`roles/minecraft_server/templates/server.properties.j2`:
```
server-port={{ minecraft_server_port }}
max-memory={{ minecraft_max_memory }}
min-memory={{ minecraft_min_memory }}
```

`roles/minecraft_server/templates/minecraft.service.j2`:
```
[Unit]
Description=Minecraft Server
After=network.target

[Service]
WorkingDirectory={{ minecraft_server_path }}
ExecStart=/usr/bin/screen -DmS minecraft /usr/bin/java -Xmx{{ minecraft_max_memory }} -Xms{{ minecraft_min_memory }} -jar server.jar nogui

[Install]
WantedBy=multi-user.target
```

`roles/minecraft_server/handlers/main.yml`:
```yaml
---
- name: restart minecraft
  systemd:
    name: minecraft
    state: restarted
```

Now, let's create a playbook to use this role:

`playbooks/minecraft_server.yml`:
```yaml
---
- hosts: raspberrypi
  become: yes

  roles:
    - minecraft_server
```

Finally, update the `inventory/group_vars/raspberrypi.yml` file with any necessary variables for your Raspberry Pi hosts.

You can now run the playbook:

```bash
ansible-playbook -i inventory/production playbooks/minecraft_server.yml
```

This will set up a Minecraft server on your Raspberry Pi hosts. The server will start automatically on system boot, and you can manage it using systemd commands like `sudo systemctl stop minecraft` and `sudo systemctl start minecraft`.
~~~

</details>

### Model E: Google AI Studio - Gemini 1.5 Pro

<details>
  <summary>Click to expand Model E's response</summary>
  
~~~
Homelab Project: Monitoring with Prometheus & Grafana
This project will set up Prometheus and Grafana on an Ubuntu server, allowing you to monitor various system metrics and create insightful dashboards.
Playbook Structure:
Based on your existing Ansible directory structure, we'll add a new playbook and role for this project:
Playbook: playbooks/prometheus_grafana_deploy/deploy.yml
Role: roles/prometheus_grafana
Role Structure:
The prometheus_grafana role will follow the standard Ansible role structure:
roles/prometheus_grafana/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
│   ├── prometheus.yml.j2
│   └── grafana_datasource.yml.j2
└── vars
    └── main.yml

Implementation Steps:
Define Variables: In roles/prometheus_grafana/vars/main.yml, define variables like Prometheus version, Grafana version, download URLs, installation paths, etc.
Download & Install Packages: In roles/prometheus_grafana/tasks/main.yml, use the apt module to install required packages like prometheus, prometheus-node-exporter, and grafana.

Configure Prometheus:
Use the template module to create the Prometheus configuration file (prometheus.yml) from the template prometheus.yml.j2.
Customize the template to include jobs for scraping metrics from the server itself and any other targets you want to monitor (e.g., other servers, network devices).
Start and enable the Prometheus service.

Configure Grafana:
Use the template module to create a Grafana datasource configuration file (grafana_datasource.yml) from the template grafana_datasource.yml.j2.
Customize the template with the URL and access details for Prometheus.
Use the grafana_datasource module to add the Prometheus datasource to Grafana.
Start and enable the Grafana service.

Playbook Content:
The playbooks/prometheus_grafana_deploy/deploy.yml playbook will be simple:
---
- hosts: monitoring_servers
  become: true
  roles:
    - prometheus_grafana

Inventory Configuration:
Create a group in your inventory file (e.g., monitoring_servers) and add the target server's hostname or IP address to this group.

Running the Playbook:
ansible-playbook -i inventory/production/hosts playbooks/prometheus_grafana_deploy/deploy.yml

Additional Considerations:
Security: Configure user authentication and access control for both Prometheus and Grafana.
Dashboards: Create Grafana dashboards to visualize the collected metrics. You can import pre-built dashboards or create your own.
Alerting: Set up alerting rules in Prometheus to get notified about potential issues.

Benefits:
This project provides a solid foundation for monitoring your homelab infrastructure. You'll have real-time insights into system performance, resource usage, and potential problems, allowing you to proactively manage your environment.
~~~

</details>

### Model F: Llama 3

<details>
  <summary>Click to expand Model F's response</summary>
  
~~~
I'll help you write an Ansible playbook and accompanying files for a fun homelab project. Let's create a "Home Media 
Server" project.

**Project Structure:**

Create a new directory called `home_media_server` inside your `playbooks` directory:
```bash
mkdir playbooks/home_media_server
```
Inside the `home_media_server` directory, create the following files and directories:

* `playbook.yml`: The main playbook file.
* `roles`: A directory to store reusable Ansible roles.
        + `media_server`: A role for configuring a media server (e.g., Plex or Kodi).
        + `storage`: A role for setting up storage devices (e.g., NAS or external hard drives).

**playbook.yml:**
```yaml
---
- name: Home Media Server
  hosts: all
  become: yes

  roles:
    - media_server
    - storage
```
This playbook defines a single play that applies to all hosts in the `all` group. It includes two roles: 
`media_server` and `storage`.

**roles/media_server/tasks/main.yml:**
```yaml
---
- name: Install media server software
  apt:
    name: plexmediaserver
    state: present

- name: Configure media server settings
  template:
    src: templates/plex_config.j2
    dest: /etc/plex/config.json
    mode: '0644'

- name: Start media server service
  service:
    name: plexmediaserver
    state: started
    enabled: yes
```
This role installs Plex Media Server, configures it using a template file, and starts the service.

**roles/media_server/templates/plex_config.j2:**
```jinja2
{
  "transcoder": {
    "ffmpeg": true,
    "ffprobe": true
  }
}
```
This template file defines some basic Plex configuration settings.

**roles/storage/tasks/main.yml:**
```yaml
---
- name: Install storage software (e.g., Samba)
  apt:
    name: samba
    state: present

- name: Configure storage shares
  template:
    src: templates/smb.conf.j2
    dest: /etc/samba/smb.conf
    mode: '0644'

- name: Start storage service (e.g., Samba)
  service:
    name: samba
    state: started
    enabled: yes
```
This role installs Samba, configures it using a template file, and starts the service.

**roles/storage/templates/smb.conf.j2:**
```jinja2
[global]
workgroup = HomeMediaServer

[share]
path = /mnt/media
name = MediaShare
comment = Media Share for Home Media Server
```
This template file defines some basic Samba configuration settings.

That's it! This playbook and roles should give you a solid foundation for setting up a home media server with Plex and
storage devices. Feel free to{"function":"update_slots","level":"INFO","line":1595,"msg":"slot context shift","n_cache_tokens":2048,"n_ctx":2048,"n_discard":1011,"n_keep":24,"n_left":2023,"n_past":2047,"n_system_tokens":0,"slot_id":0,"task_id":0,"tid":"126467037587008","timestamp":1715220167}
 modify or add more tasks as needed.

Remember to create the `templates` directory inside each role directory (e.g., `media_server/templates/`) to store 
your template files.
~~~

</details>

### Model G: Llama 3:70B

<details>
  <summary>Click to expand Model G's response</summary>
  
~~~
What a delightfully organized Ansible directory structure!

Let's create a new project together. How about we deploy a simple NextCloud instance on an Ubuntu server?

Here's the plan:

1. Create a new playbook and accompanying role for NextCloud deployment.
2. Define the necessary tasks, handlers, and templates for the deployment.
3. Configure the playbook to use the existing `ubuntu_server_setup` role as a dependency.

Let's get started!

**Create a new playbook:**

In the `playbooks` directory, create a new folder called `nextcloud_deploy` with a single file `deploy_nextcloud.yml`.
```yaml
# playbooks/nextcloud_deploy/deploy_nextcloud.yml

---
- name: Deploy NextCloud on Ubuntu Server
  hosts: ubuntu_servers
  become: true

  roles:
    - ubuntu_server_setup
    - nextcloud_deploy
```
**Create a new role:**

In the `roles` directory, create a new folder called `nextcloud_deploy` with the following subdirectories and files:
```bash
# roles/nextcloud_deploy/

defaults/
main.yml

files/
nextcloud.conf

handlers/
main.yml

meta/
main.yml

tasks/
main.yml
install_nextcloud.yml
configure_nextcloud.yml

templates/
nextcloud.conf.j2

vars/
main.yml
```
**Define the tasks and handlers:**

In `roles/nextcloud_deploy/tasks/main.yml`, define the main task list:
```yaml
# roles/nextcloud_deploy/tasks/main.yml

---
- name: Deploy NextCloud on Ubuntu Server
  tasks:
    - install_nextcloud
    - configure_nextcloud
```
In `roles/nextcloud_deploy/tasks/install_nextcloud.yml`, define the installation task:
```yaml
# roles/nextcloud_deploy/tasks/install_nextcloud.yml

---
- name: Install NextCloud
  apt:
    name: nextcloud
    state: present
```
In `roles/nextcloud_deploy/tasks/configure_nextcloud.yml`, define the configuration task:
```yaml
# roles/nextcloud_deploy/tasks/configure_nextcloud.yml

---
- name: Configure NextCloud
  template:
    src: templates/nextcloud.conf.j2
    dest: /etc/nextcloud/config.php
    mode: '0644'
  notify: restart nextcloud
```
In `roles/nextcloud_deploy/handlers/main.yml`, define the handler to restart the NextCloud service:
```yaml
# roles/nextcloud_deploy/handlers/main.yml

---
- name: Restart NextCloud Service
  service:
    name: nextcloud
    state: restarted
```
** Define the templates and variables:**

In `roles/nextcloud_deploy/templates/nextcloud.conf.j2`, define the template for the NextCloud configuration file:
```jinja2
# roles/nextcloud_deploy/templates/nextcloud.conf.j2

<?php
  $CONFIG = array (
    'instanceid' => '{{ instance_id }}',
    'passwordsalt' => '{{ password{"function":"update_slots","level":"INFO","line":1595,"msg":"slot context shift","n_cache_tokens":2048,"n_ctx":2048,"n_discard":1011,"n_keep":24,"n_left":2023,"n_past":2047,"n_system_tokens":0,"slot_id":0,"task_id":0,"tid":"126465594750528","timestamp":1715221308}
_salt }}',
    // ...
  );
?>
```
In `roles/nextcloud_deploy/vars/main.yml`, define the variables for the template:
```yaml
# roles/nextcloud_deploy/vars/main.yml

---
instance_id: "myNextCloudInstance"
password_salt: "MySuperSecretPasswordSalt"
```
That's it! You now have a new playbook and role to deploy a NextCloud instance on an Ubuntu server. Run the playbook 
using `ansible-playbook -i <inventory_file> playbooks/nextcloud_deploy/deploy_nextcloud.yml` to see the magic happen.

How was that? Would you like to add any other features or modifications to this project?
~~~

</details>

### Model H: Mistral Large

<details>
  <summary>Click to expand Model H's response</summary>
  
~~~
Based on your current Ansible directory structure, I suggest a fun homelab project to set up a self-hosted GitLab instance on a dedicated server in your homelab. This will allow you to manage your code and collaborate with others in a private setting. Here's how you can structure your Ansible files for this project:

1. Create a new directory for the GitLab project inside the `playbooks` directory:

```
mkdir -p playbooks/gitlab_install
```

2. Create a new playbook file for the GitLab installation inside the newly created directory:

```
touch playbooks/gitlab_install/install_gitlab.yml
```

3. Edit the `install_gitlab.yml` file and add the following content:

```yaml
---
- name: Install GitLab
  hosts: gitlab
  become: true
  roles:
    - common
    - gitlab_install
```

4. Create a new role directory for the GitLab installation inside the `roles` directory:

```
mkdir -p roles/gitlab_install/{tasks,templates,vars,defaults,files,handlers,meta}
```

5. Create a `main.yml` file inside the `tasks` directory of the new role:

```
touch roles/gitlab_install/tasks/main.yml
```

6. Edit the `main.yml` file and add the following content:

```yaml
---
- name: Include GitLab official repository
  apt_repository:
    repo: 'deb https://packages.gitlab.com/gitlab/gitlab-ee/ {{ ansible_lsb.codename }}-ee main'
    state: present
    filename: gitlab-ee

- name: Add GitLab package signing key
  apt_key:
    url: https://packages.gitlab.com/gpg.key
    state: present

- name: Install GitLab package
  apt:
    name: gitlab-ee
    state: present

- name: Configure GitLab
  template:
    src: gitlab.rb.j2
    dest: /etc/gitlab/gitlab.rb
  notify:
    - Restart GitLab

- name: Run GitLab reconfigure
  command: gitlab-ctl reconfigure
  changed_when: false

```

7. Create a `gitlab.rb.j2` file inside the `templates` directory of the new role:

```
touch roles/gitlab_install/templates/gitlab.rb.j2
```

8. Edit the `gitlab.rb.j2` file and add the following content (you can customize the configuration as needed):

```ruby
external_url 'http://gitlab.yourdomain.com'
gitlab_rails['lfs_enabled'] = true
gitlab_rails['time_zone'] = 'UTC'
gitlab_rails['gitlab_shell_ssh_port'] = 22
nginx['redirect_http_to_https'] = true
```

9. Create a `main.yml` file inside the `handlers` directory of the new role:

```
touch roles/gitlab_install/handlers/main.yml
```

10. Edit the `main.yml` file and add the following content:

```yaml
---
- name: Restart GitLab
  command: gitlab-ctl restart
  listen: "Restart GitLab"
```

11. Add the new GitLab server to your inventory file (e.g., `inventory/staging/hosts` or `inventory/production/hosts`):

```
[gitlab]
gitlab.yourdomain.com
```

12. Update your `ansible.cfg` file to include the new role's directory if it's not already present:

```
[defaults]
roles_path = ./roles
```

Now you can run the GitLab installation playbook with the following command:

```
ansible-playbook playbooks/gitlab_install/install_gitlab.yml -i inventory/staging/hosts
```

Or for production:

```
ansible-playbook playbooks/gitlab_install/install_gitlab.yml -i inventory/production/hosts
```

This will set up a self-hosted GitLab instance on your dedicated server. You can further customize the configuration by modifying the `gitlab.rb.j2` file and adding or modifying variables in the `vars` and `defaults` directories of the new role.
~~~

</details>

## Broader Analysis of All Tested Models

### Model A: im-a-good-gpt2-chatbot

Model A created a comprehensive media server setup using Docker, Docker Compose, and popular applications like Plex, Sonarr, Radarr, and Transmission. The response was well-organized, adhered to Ansible best practices, and included a detailed playbook, role, inventory adjustments, and execution instructions.

### Model B: im-also-a-good-gpt2-chatbot

Model B designed a Grafana monitoring stack integrated with Prometheus for system metrics collection. While well-structured, Model B's response didn't quite match the level of detail and organization provided by Model A. However, the inclusion of the node_exporter role was a nice touch. Also of note

### Comparison to Models C, D, E, F, G, and H

Models C (ChatGPT - GPT-4) and D (Claude 3 Opus) focused on setting up a Minecraft server, providing thorough and well-structured responses similar to Model A. They followed Ansible best practices and included all the necessary components for a complete deployment.

Model E (Gemini 1.5 Pro) provided a high-level overview of setting up a monitoring stack with Prometheus and Grafana, similar to Model B. However, Model E's response lacked the depth and detail present in Model B's response.

Models F (Llama 3) and G (Llama 3:70B) focused on media server projects, similar to Model A. While their responses were well-structured and included the necessary components, they didn't quite match the level of detail and organization found in Model A's response.

Model H (Mistral Large) proposed setting up a self-hosted GitLab instance, which was a different project compared to Models A and B. However, the response was comprehensive, well-structured, and followed Ansible best practices, similar to Model A.

## Final Thoughts

I attempted to compare the responses from Models C, D, E, F, G, and H to the original Models A and B. At this point, I haven't actually attempted to deploy any of these Ansible playbooks, but plan to follow-up after attempting to deploy all of the suggestions.

Model A (im-a-good-gpt2-chatbot) stands out for its comprehensive and well-organized response, providing a detailed media server setup using Docker and popular applications. It set a high standard for the other models to follow.

Models C (ChatGPT - GPT-4), D (Claude 3 Opus), and H (Mistral Large) came close to matching the quality and comprehensiveness of Model A, providing well-structured and detailed responses for their respective projects.

Model B (im-also-a-good-gpt2-chatbot), while not as comprehensive as Model A, still provided a solid monitoring stack setup. Models F (Llama 3) and G (Llama 3:70B) also delivered well-structured media server projects, although not as detailed as Model A.

Model E (Gemini 1.5 Pro) fell short compared to both Model A and B, providing only a high-level overview of the monitoring stack setup without the necessary depth and detail.

In conclusion, Model A (im-a-good-gpt2-chatbot) demonstrated the strongest performance, followed closely by Models C, D, and H. Models B, F, and G provided solid responses but didn't quite match the level of detail and organization found in Model A. Model E lagged behind the others, offering a less comprehensive solution compared to both Model A and B.

It's important to note that this analysis is based on a single-shot prompt, meaning that each model was provided with the prompt only once and generated its response accordingly. With further refinement and iteration, it's likely that the models could have generated even better results. Fine-tuning the prompt, providing additional context, or engaging in a multi-turn conversation could potentially lead to more comprehensive and tailored Ansible playbooks from all the models involved.
