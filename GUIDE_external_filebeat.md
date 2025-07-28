# Guide: Installing Filebeat on an External Production Server

This guide explains how to install and configure Filebeat on an external production server so it can forward logs to your central ELK stack (Elasticsearch, Logstash, Kibana) for ingestion and visualization.

---

## Prerequisites

- **Central ELK Stack**: Already running and accessible from the production server (Logstash listening on port 5044).
- **Production Server**: Linux server with root or sudo access.
- **Network**: Ensure the production server can reach the ELK server on port 5044 (Logstash Beats input).

---

## 1. Install Filebeat and Metricbeat on the Production Server

### For Debian/Ubuntu:

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates gnupg curl wget

# Add Elastic GPG key
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg

# Add Elastic 9.x APT repository
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

# Install Filebeat and Metricbeat
sudo apt-get update
sudo apt-get install -y filebeat metricbeat
```

Or use your package manager as described in the [official Filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html) and [official Metricbeat documentation](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation.html).

---

## 2. Configure Filebeat to Forward Logs to Logstash

Edit the Filebeat configuration file (usually `/etc/filebeat/filebeat.yml`):

- **Disable Elasticsearch output** (comment out or remove):

```yaml
#output.elasticsearch:
#  hosts: ["localhost:9200"]
```

- **Enable Logstash output** and set your ELK server address:

```yaml
output.logstash:
  hosts: ["<ELK_SERVER_IP_OR_HOSTNAME>:5044"]
```
To use SSL, make sure to copy the certs from the Elastic server and paste them into the path "/etc/filebeat/certs/", then use the following config:
```yaml
output.logstash:
  hosts: ["<ELK_SERVER_IP_OR_HOSTNAME>:5044"]

  protocol: "https"
  ssl.verification_mode: none
  ssl.certificate: "/etc/filebeat/certs/http.crt"
  ssl.key: "/etc/filebeat/certs/http.key"
```

- **Specify which logs to ship** (default is `/var/log/*.log`, adjust as needed):

```yaml
filebeat.inputs:
  - type: filestream
    id: backend-app
    enabled: true
    paths:
      - /opt/pricing/ssl/pricing_backend/Log_backend.log
    fields:
      log_type: backend_logs
    fields_under_root: true
    parsers:
      - multiline:
          type: pattern
          pattern: '^\d{4}/\d{2}/\d{2} \d{2}:\d{2}:\d{2}'
          negate: true
          match: after

  - type: filestream
    id: nginx-access
    enabled: true
    paths:
      - /var/log/nginx/access.log
    fields:
      log_type: nginx_access
    fields_under_root: true

  - type: filestream
    id: nginx-error
    enabled: true
    paths:
      - /var/log/nginx/error.log
    fields:
      log_type: nginx_error
    fields_under_root:
      true
      # Add more log files as needed
```

---

## 3. Start and Enable Filebeat

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

Check status:

```bash
sudo systemctl status filebeat
```

---

## 4. Verify Log Forwarding

- On the filebeat server, check the output of the following command:

```bash
sudo filebeat test output
```

If the connection to Logstash is set up correctly, the command will show ok

- On the ELK server, check Logstash logs to confirm it is receiving data.
- In Kibana, create or update an index pattern (e.g., `filebeat-*`) and check the **Discover** tab for incoming logs from the production server.

---

## 5. Troubleshooting

- Ensure firewalls allow outbound traffic from the production server to the ELK server on port 5044.
- Check Filebeat logs (`/var/log/filebeat/filebeat.log`) for errors.
- Confirm Logstash is running and listening on port 5044 using:
```bash
sudo journalctl -u logstash -f
```

---
