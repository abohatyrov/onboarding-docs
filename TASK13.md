# Deploying Alertmanager with Telegram Integration

## Introduction

Alertmanager is a crucial component in the Prometheus ecosystem, responsible for handling and sending alerts based on defined rules. In this guide, we will deploy Alertmanager on a virtual machine (VM) and configure it to send notifications to a Telegram channel in case of critical alerts.

### Prerequisites

1. **VM Setup**: Ensure you have a VM running, as mentioned in the previous documentation.

2. **Ansible Role**: We will use a new Ansible role for deploying Alertmanager, as provided in the script.

3. **Telegram Bot Token and Chat ID:** Obtain a Telegram Bot Token and Chat ID for configuring Telegram notifications.

### Creating a Telegram Bot
To set up a Telegram bot, follow these steps:
1. Open the Telegram app and search for the "BotFather" bot.
2. Start a chat with the BotFather and use the `/newbot` command to create a new bot.
3. Follow the instructions from the BotFather to choose a name and username for your bot.
4. Once the bot is created, the BotFather will provide you with a unique bot token. Save this token; you'll need it later.

### Finding Chat ID
To find the Chat ID for configuring Telegram notifications, follow these steps:

1. Start a chat with your newly created bot on Telegram.
2. Send a message to the bot.
3. Open a web browser and go to the following URL, replacing BOT_TOKEN with your bot's token:
```
https://api.telegram.org/botBOT_TOKEN/getUpdates
```
For example:
```
https://api.telegram.org/bot123456789:ABCDE_fghijklmnopqrstuvwxyz/getUpdates
```
4. Look for the "chat" object in the response. The "id" field within the "chat" object is your Chat ID. Save this Chat ID; you'll use it in the Alertmanager configuration.

## Step-by-Step Guide

### 1. Update Prometheus Configuration

Add the following configuration to your `prometheus.yml` file to specify the Alertmanager endpoint:

```yaml
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']
```

This configuration tells Prometheus to send alerts to the Alertmanager running on the specified endpoint.

### 2. Create Alertmanager Ansible Role

Use the provided Ansible role to install, configure, and manage permissions for Alertmanager. The role consists of tasks for installing Alertmanager, configuring it with custom settings, and creating necessary permissions.

#### Example Role Structure:

```plaintext
alertmanager_role/
├── tasks/
│   ├── install.yml
│   ├── configure.yml
│   ├── permissions.yml
│   ├── service.yml
├── templates/
│   ├── alert.rules.yml.j2
│   ├── alertmanager.service.j2
│   ├── alertmanager.tmpl.j2
│   ├── alertmanager.yml.j2
```

### 3. Configure Alertmanager

The configuration tasks (`configure.yml`) include creating directories, copying binaries, and setting up configuration files. The provided templates (`alert.rules.yml.j2`, `alertmanager.service.j2`, `alertmanager.tmpl.j2`, `alertmanager.yml.j2`) define alert rules, service settings, Telegram templates, and the main Alertmanager configuration.

### Tasks:

1. **Install Alert Manager (`install.yml`):**

   - **Download and Extract Alertmanager:**
     - Downloads the Alertmanager binary from the specified release.
     - Extracts the binary to `/tmp/`.
     - Sets ownership and permissions.

   - **Copy Alertmanager binary:**
     - Copies the Alertmanager binary to `/usr/local/bin/`.
     - Sets ownership and permissions.

   - **Copy amtool binary:**
     - Copies the `amtool` binary to `/usr/local/bin/`.
     - Sets ownership and permissions.

   #### Example (`install.yml`):

   ```yaml
   - name: Download and Extract Alertmanager
     unarchive:
       src: https://github.com/prometheus/alertmanager/releases/download/v{{ alert_manager_version }}/alertmanager-{{ alert_manager_version }}.linux-amd64.tar.gz
       dest: /tmp/
       owner: "{{ alert_manager_user }}"
       group: "{{ alert_manager_group }}"
       mode: 0755
       remote_src: true

   - name: Copy alertmanager binary
     copy:
       src: /tmp/alertmanager-{{ alert_manager_version }}.linux-amd64/alertmanager
       dest: /usr/local/bin/alertmanager
       owner: "{{ alert_manager_user }}"
       group: "{{ alert_manager_group }}"
       mode: 0755
       remote_src: yes

   - name: Copy amtool binary
     copy:
       src: /tmp/alertmanager-{{ alert_manager_version }}.linux-amd64/amtool
       dest: /usr/local/bin/amtool
       owner: "{{ alert_manager_user }}"
       group: "{{ alert_manager_group }}"
       mode: 0755
       remote_src: yes
   ```

2. **Configure Alert Manager (`configure.yml`):**

   - **Create Alertmanager configuration directory:**
     - Creates the `/etc/alertmanager` directory.

   - **Create Alertmanager configuration file:**
     - Creates the main Alertmanager configuration file from the provided template.
     - Notifies to restart the Alert Manager service.

   - **Create Alertmanager rules file:**
     - Creates the alert rules file from the provided template.
     - Notifies to restart the Alert Manager service.

   - **Create Alertmanager templates directory:**
     - Creates the `/etc/alertmanager/templates` directory.

   - **Create Alertmanager template:**
     - Creates the Telegram template from the provided template.
     - Notifies to restart the Alert Manager service.

   #### Example (`configure.yml`):

   ```yaml
   - name: Create Alertmanager configuration directory
     file:
       path: /etc/alertmanager
       state: directory
     notify: Restart Alert Manager service

   - name: Create Alertmanager configuration file
     template:
       src: alertmanager.yml.j2
       dest: /etc/alertmanager/config.yml
     notify: Restart Alert Manager service

   - name: Create Alertmanager rules file
     template:
       src: alert.rules.yml.j2
       dest: /etc/prometheus/alert.rules.yml
     notify: Restart Alert Manager service

   - name: Create Alertmanager templates directory
     file:
       path: /etc/alertmanager/templates
       state: directory
     notify: Restart Alert Manager service

   - name: Create Alertmanager template
     template:
       src: alertmanager.tmpl.j2
       dest: /etc/alertmanager/templates/alertmanager.tmpl
     notify: Restart Alert Manager service
   ```

3. **Configure Permissions (`permissions.yml`):**

   - **Add Alert Manager group:**
     - Adds the specified group for Alert Manager.

   - **Add Alert Manager user:**
     - Adds the specified user for Alert Manager, adding it to the group.
     - Sets the shell and home directory.

   #### Example (`permissions.yml`):

   ```yaml
   - name: Add Alert Manager group
     group:
       name: "{{ alert_manager_group }}"
       state: present

   - name: Add Alert Manager user
     user:
       name: "{{ alert_manager_user }}"
       groups: ["{{ alert_manager_group }}", wheel]
       shell: /bin/false
       createhome: no
   ```

4. **Create Alert Manager Service (`service.yml`):**

   - **Copy Alert Manager service configuration:**
     - Creates the Alert Manager service configuration file from the provided template.
     - Notifies to enable and restart the Alert Manager service.

   #### Example (`service.yml`):

   ```yaml
   - name: Copy Alert Manager service configuration
     template:
       src: alertmanager.service.j2
       dest: /etc/systemd/system/alertmanager.service
     notify: 
       - Enable Alert Manager service
       - Restart Alert Manager service
   ```

### Templates:

1. **Alert Rules Template (`alert.rules.yml.j2`):**

   - Defines Prometheus alert rules in YAML format.

   #### Example (`alert.rules.yml.j2`):

   ```yaml
   groups:
   - name: alert.rules
     rules:
     - alert: InstanceDown
       expr: up == 0
       for: 15s
       labels:
         severity: "critical"
       annotations:
         summary: "Endpoint {{ $labels.instance }} down"
         description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."
   ```

2. **Alert Manager Service Template (`alertmanager.service.j2`):**

   - Defines the systemd service configuration for Alert Manager.

   #### Example (`alertmanager.service.j2`):

   ```ini
   [Unit]
   Description=Alertmanager service
   After=network.target

   [Service]
   User={{ alert_manager_user }}
   Group={{ alert_manager_group }}
   WorkingDirectory=/etc/alertmanager
   ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/config.yml

   [Install]
   WantedBy=multi-user.target
   ```

3. **Alert Manager Telegram Template (`alertmanager.tmpl.j2`):**

   - Defines the Telegram message template for Alert Manager notifications.

   #### Example (`alertmanager.tmpl.j2`):

   ```raw
   {{ define "telegram.default.text" }}
   {{ range .Alerts }}
   <b>🔥{{ .Status | toUpper }}</b> - {{ .Annotations.summary }} on {{ .Labels.instance }}
   <b>Details:</

b>
   {{ .Annotations.description }}

   {{ end }}
   {{ end }}
   ```

4. **Alert Manager Main Configuration Template (`alertmanager.yml.j2`):**

   - Defines the main Alert Manager configuration in YAML format, including routes, receivers, and templates.

   #### Example (`alertmanager.yml.j2`):

   ```yaml
   route:
       receiver: telegram

   receivers:
     - name: 'telegram'
       telegram_configs:
       - bot_token: {{ telegram_api_token }}
         api_url: https://api.telegram.org
         chat_id: {{ telegram_chat_id }}
         parse_mode: 'HTML'
         message: '{% raw %}{{ template "telegram.default.text" . }}{% endraw %}'

   templates:
     - '/etc/alertmanager/templates/alertmanager.tmpl'
   ```

These tasks and templates collectively allow you to deploy Alertmanager, configure it, and integrate it with Telegram for timely notifications based on defined alert rules.

### 4. Run Ansible Playbook

Add new role to the existing Prometheus playbook from task 6.
```yaml
- name: Install and Configure Prometheus
  hosts: prometheus
  become: 'yes'

  roles:
    - prometheus
    - alert_manager
```

Execute the Ansible playbook to deploy Alertmanager and apply the configuration:

```bash
ansible-playbook prometheus.yml
```

### 5. Verify Alertmanager Installation

Check the status of the Alertmanager service to ensure it is running:

```bash
systemctl status alertmanager
```

### 6. Test Alert Rule

Create a test alert rule in the `alert.rules.yml` file, or use the provided example for monitoring instance availability. The rule triggers an alert if an instance is down for more than 15 seconds.

#### Example `alert.rules.yml`:

```yaml
groups:
- name: alert.rules
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 15s
    labels:
      severity: "critical"
    annotations:
      summary: "Endpoint {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."
```

### 7. Restart Alertmanager

Restart the Alertmanager service to apply the new configuration:

```bash
systemctl restart alertmanager
```

### 8. Verify Telegram Notification

Trigger the test alert rule or wait for an actual alert, and verify that a notification is sent to your Telegram channel.

## Conclusion

Congratulations! You have successfully deployed Alertmanager and configured it to send notifications to Telegram. This setup enhances your monitoring capabilities, ensuring timely awareness of critical events. Feel free to explore additional configurations and customize alert rules based on your specific requirements.