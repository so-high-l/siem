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
- Local development with Vagrant (Ubuntu 20.04)

---

## 🚀 Quick Start

### For External Ubuntu Servers

1. **Generate the ssh key for ansible to use**

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/siem_id_rsa
```

This creates:

- Private key: ~/.ssh/siem_id_rsa
- Public key: ~/.ssh/siem_id_rsa.pub

2. **Edit inventories/hosts.ini**

Add the private ssh key generated in step 1
Example:
host_name ansible_host=external-server-ip ansible_user=user ansible_ssh_private_key_file=~/.ssh/siem_id_rsa

3. **Set variables and secrets:**
   Needed variables to run the setup:

- elastic_password
- kibana_system_password
- logstash_system_password
- kibana_encryption_key
- logstash_password
- smtp_password

```bash
# Command to encrypt a variable
ansible-vault encrypt_string 'your_secret_value' --name your_variable_name
```

- Store secrets in `group_vars/all/vault.yml` (encrypt with Ansible Vault)

4. **Copy the ssh public key to remote server**

   - 4.1 Option A:

   ```bash
   # Use ssh-copy-id - Recommended:
   ssh-copy-id -i ~/.ssh/siem_id_rsa.pub user@external-server-ip
   ```

   - 4.2 Option B - Manual method:

   ```bash
   # Show the public key
   cat ~/.ssh/siem_id_rsa.pub

   # On the remote server:
   ## If ssh is not configured
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh

   ## Paste the public key here
   echo "ssh-rsa AAAAB3Nza... your_email@example.com" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

5. **Test Connection with Ansible**

   ```bash
   # Test the connection with Ansible to the external server
   ansible -i inventories/hosts.ini elk_host -m ping
   ```

   ```bash
   # You should see
   external-server | SUCCESS => {
    "changed": false,
    "ping": "pong"
   }
   ```

6. **Run the SIEM Playbook**
The --ask-become-pass is used to pass the sudo password for tasks that need sudo previleges 
The --ask-vault-pass is used to pass the vault password that was used during enryption of secrets

   ```bash
   # Run complete deployment
   ansible-playbook -i inventories/hosts.ini playbook.yml --ask-vault-pass --ask-become-pass
   ```

   ```bash
   # Or run individual components
   ansible-playbook playbook.yml --ask-vault-pass --ask-become-pass --tags elasticsearch
   ansible-playbook playbook.yml --ask-vault-pass --ask-become-pass --tags kibana
   ansible-playbook playbook.yml --ask-vault-pass --ask-become-pass --tags logstash
   ```

### 🏷️ Using Tags for Selective Deployment

See `TAGS_USAGE_GUIDE.md` for detailed information on running specific components:

```bash
# Deploy only Elasticsearch
ansible-playbook playbook.yml -i inventories/hosts.ini --ask-vault-pass --ask-become-pass --tags elasticsearch

# Deploy only Kibana
ansible-playbook playbook.yml -i inventories/hosts.ini --ask-vault-pass --ask-become-pass --tags kibana

# Deploy multiple components
ansible-playbook playbook.yml -i inventories/hosts.ini --ask-vault-pass --ask-become-pass --tags "elasticsearch,kibana,logstash"

# Run system setup only
ansible-playbook playbook.yml -i inventories/hosts.ini --ask-vault-pass --ask-become-pass --tags setup
```

---

**Security:**

- All internal communication uses certificate-based HTTPS (TLS)
- Internal CA is generated and trusted by all components
- User roles: `elastic`, `kibana_system`, `logstash_writer`, etc.

---

## 📦 3. Ansible Roles Breakdown

### `roles/elasticsearch`

- Installs Elasticsearch 9.0.0
- Configures TLS for HTTP and transport
- Generates internal CA and distributes certs
- Creates default users and roles for secure access
- **Ubuntu 22.04 compatible** with optimized Java 17 configuration

### `roles/kibana`

- Installs Kibana 9.0.0
- Configures secure connection to Elasticsearch
- Waits for Elasticsearch readiness
- Optionally imports prebuilt dashboards
- **Ubuntu 22.04 compatible** with proper service configuration

### `roles/logstash`

- Installs and configures Logstash 9.0.0
- Sets up pipelines to receive from Beats and output to Elasticsearch
- Uses secure certificates for communication
- **Ubuntu 22.04 compatible** with updated repository configuration

### `roles/beats` (Filebeat & Metricbeat)

- Installs Filebeat and Metricbeat 9.0.0
- Configures modules and output to Logstash
- Uses CA certs for secure communication
- **Ubuntu 22.04 compatible** with proper dependency management

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
   git clone https://github.com/so-high-l/siem
   cd siem
   ```
2. **Edit inventory:**
   - Edit `inventories/hosts.ini` and add your server:
     ```ini
     [siem]
     your.server.ip ansible_user=youruser ....
     ```
3. **Set variables and secrets:**
   Needed variables to run the setup:

- elastic_password
- kibana_system_password
- logstash_system_password
- kibana_encryption_key
- logstash_password
- smtp_password

Command to encrypt a variable

```bash
# ansible-vault encrypt_string '...............' --name 'logstash_system_password'
ansible-vault encrypt_string 'your_secret_value' --name your_variable_name
```

- Store secrets in `group_vars/all/vault.yml` (encrypt with Ansible Vault):

4. **Run the playbook:**

- If using vault:
  ```bash
  ansible-playbook --ask-vault-pass --ask-become-pass -i inventories/hosts.ini playbook.yml
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
sudo systemctl restart metricbeat
sudo systemctl restart filebeat
sudo systemctl restart elastalert
```

---

## 📁 10. File Structure

```
elk-ansible/
├── ansible.cfg                 # Ansible configuration
├── playbook.yml               # Main playbook for standard deployment
├── inventories/
│   └── hosts.ini              # Standard inventory
├── group_vars/
│   └── all/
│       └── vault.yml          # Encrypted secrets
└── roles/
    ├── elasticsearch/          # Elasticsearch 9.0.0 role
    ├── kibana/                # Kibana 9.0.0 role
    ├── logstash/              # Logstash 9.0.0 role
    ├── beats/                 # Filebeat & Metricbeat 9.0.0 role
    └── elastalert/            # ElastAlert2 role
```

---

## 🔄 11. Version Compatibility

- **Elasticsearch:** 9.0.0
- **Kibana:** 9.0.0
- **Logstash:** 9.0.0
- **Beats:** 9.0.0
- **Ubuntu:** 20.04 LTS, 22.04 LTS
- **Java:** OpenJDK 17
- **Python:** 3.8+

---
