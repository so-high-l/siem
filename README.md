# SIEM Lab Native - Automated ELK Stack Deployment with Ansible

---

## 🏗️ 1. Overview

This project provides a fully automated deployment of a Security Information and Event Management (SIEM) lab using Ansible. It sets up a production-like Elastic Stack (ELK) with secure TLS communication, role-based access control, and alerting via ElastAlert2. The stack is ideal for security monitoring, log management, and alerting use cases.

**Technologies Used:**

- **Elasticsearch** 9.0.0
- **Kibana** 9.0.0
- **Logstash** 9.0.0
- **Beats:** Filebeat 9.0.0, Metricbeat 9.0.0
- **ElastAlert2** (latest stable)
- **Ansible** (automation)

**Deployment Targets:**

- Any external Ubuntu-based server (tested on Ubuntu 20.04/22.04)

---



**Security:**

- All internal communication uses certificate-based HTTPS (TLS)
- Internal CA is generated and trusted by all components
- User roles: `elastic`, `kibana_system`, `logstash_writer`, etc.

---

## 📦 3. Ansible Roles Breakdown

### `roles/elasticsearch`

- Installs Elasticsearch
- Configures TLS for HTTP and transport
- Generates internal CA and distributes certs
- Creates default users and roles for secure access

### `roles/kibana`

- Installs Kibana
- Configures secure connection to Elasticsearch
- Waits for Elasticsearch readiness
- Optionally imports prebuilt dashboards

### `roles/logstash`

- Installs and configures Logstash
- Sets up pipelines to receive from Beats and output to Elasticsearch
- Uses secure certificates for communication

### `roles/beats` (Filebeat & Metricbeat)

- Installs Filebeat and Metricbeat
- Configures modules and output to Logstash
- Uses CA certs for secure communication

### `roles/elastalert`

- Clones ElastAlert2 repo
- Creates Python virtualenv for isolation
- Configures alert rules and connects securely to Elasticsearch
- Sets up as a systemd service

---

## 🔐 4. Security

- **TLS Everywhere:**
  - Elasticsearch HTTP and transport use TLS
  - Certs distributed to Beats, Kibana, Logstash
- **Password Management:**
  - Uses `elasticsearch-setup-passwords` for initial user setup
  - Passwords stored securely with Ansible Vault
- **Role-Based Access Control:**
  - Only required roles/users are created
  - Principle of least privilege
- **Ansible Vault:**
  - All secrets (passwords, keys) are encrypted in `group_vars/all/vault.yml`

---

## ⚙️ 5. Installation Guide

### 🌐 On External Ubuntu Server

**Prerequisites:**

- SSH access to the target server
- Python 3 installed on the target
- Ansible installed on your control machine

**Steps:**

1. **Clone the repository:**
   ```bash
   git clone <repo-url>
   cd siem-lab-native
   ```
2. **Edit inventory:**
   - Edit `inventories/hosts.ini` and add your server:
     ```ini
     [siem]
     your.server.ip ansible_user=youruser
     ```
3. **Set variables and secrets:**
   - Edit variables in `group_vars/` as needed
   - Store secrets in `group_vars/all/vault.yml` (encrypt with Ansible Vault):
     ```bash
     ansible-vault edit group_vars/all/vault.yml
     ```
4. **Run the playbook:**

- If using vault:
  ```bash
  ansible-playbook --ask-vault-pass -i inventories/hosts.ini playbook.yml
  ```

**Tips:**

- Ensure your SSH key is set up for passwordless login, or use `--ask-pass`.
- Review all variables before running in production.

---

## ♻️ 6. Idempotency

All Ansible roles are designed to be idempotent:

- **TLS certificates** are only generated if missing
- **Passwords** are initialized once and not reset on rerun
- **Dashboards** are imported only if not already present
- **ElastAlert2 index** is created only if missing

---

## 📊 7. Dashboards

- **Exporting Dashboards:**
  - Use Kibana UI or API:
    ```bash
    curl -X POST "https://<kibana-host>:5601/api/saved_objects/_export" \
      -H "kbn-xsrf: true" \
      -u elastic:<password> \
      -d '{"type":["dashboard"]}' --output dashboard.ndjson
    ```
- **Importing Dashboards:**
  - Preconfigured dashboards are in `roles/kibana/files/dashboard.ndjson`
  - Ansible imports dashboards automatically if not present
  - Manual import via API:
    ```bash
    curl -X POST "https://<kibana-host>:5601/api/saved_objects/_import" \
      -H "kbn-xsrf: true" \
      -u elastic:<password> \
      --form file=@dashboard.ndjson
    ```

---

## 📣 8. ElastAlert2

- **Installation:**
  - Cloned to `/opt/elastalert`
  - Uses Python virtualenv for isolation
- **Configuration:**
  - Rules are templated and stored in `roles/elastalert/templates/`
  - Connects securely to Elasticsearch using CA certs
- **Idempotency:**
  - ElastAlert index is created only if missing
- **Service:**
  - Managed as a systemd service (`elastalert`)
- **Rule Templates:**
  - DDoS, login failures, server errors, SMTP auth, etc.

---

## 🧪 9. Troubleshooting

**Common Issues:**

- **Wrong credentials:**
  - Check `vault.yml` and user roles
- **TLS verification errors:**
  - Ensure all components have the correct CA cert
- **Failed service startup:**
  - Check logs: `journalctl -u <service>`

**Restarting Services:**

```bash
sudo systemctl restart elasticsearch
sudo systemctl restart kibana
sudo systemctl restart logstash
sudo systemctl restart elastalert
```

**Testing Endpoints:**

- **Elasticsearch:**
  ```bash
  curl -k -u elastic:<password> https://<es-host>:9200/
  ```
- **Kibana:**
  ```bash
  curl -k -u elastic:<password> https://<kibana-host>:5601/api/status
  ```
- **Logstash:**
  - Check logs: `sudo journalctl -u logstash`

---

## 📁 10. File/Directory Structure

```
siem-lab-native/
├── ansible.cfg                # Ansible configuration
├── group_vars/                # Group and global variables
│   └── all/
│       └── vault.yml          # Encrypted secrets (Ansible Vault)
├── inventories/
│   └── hosts.ini              # Inventory file
├── playbook.yml               # Main playbook
├── roles/                     # Ansible roles
│   ├── elasticsearch/         # Elasticsearch role
│   ├── kibana/                # Kibana role
│   ├── logstash/              # Logstash role
│   ├── beats/                 # Filebeat & Metricbeat role
│   └── elastalert/            # ElastAlert2 role
├── Vagrantfile                # (Optional) For local VM testing
└── README.md                  # This file
```

**Key Files:**

- `playbook.yml`: Entry point for Ansible
- `group_vars/all/vault.yml`: Encrypted secrets
- `roles/*/tasks/`: Main automation logic for each component
- `roles/kibana/files/dashboard.ndjson`: Prebuilt dashboards

---
