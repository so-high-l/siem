ElastAlert Ansible Role
======================

This role installs and configures [ElastAlert] for alerting on Elasticsearch data. It provides example detection rules for common security and operational events, and supports email alerting via SMTP.

---

Requirements
------------
- Ansible 2.9+
- A running Elasticsearch instance accessible to ElastAlert
- Python 3 on the target host

---

Role Variables
--------------
The following variables can be set to customize ElastAlert and SMTP alerting:

- `elastalert_version`: Version of ElastAlert to install (default: v2.25.0)
- `elastalert_dir`: Install directory for ElastAlert
- `elastalert_virtualenv`: Path to the Python virtualenv
- `elastalert_config_file`: Path to the main ElastAlert config
- `elastalert_rules_dir`: Directory for custom rules
- `elasticsearch_host`: Elasticsearch hostname (default: localhost)
- `elasticsearch_port`: Elasticsearch port (default: 9200)
- `smtp_host`: SMTP server hostname (default: smtp.gmail.com)
- `smtp_port`: SMTP server port (default: 587)
- `smtp_user`: SMTP username (email address)
- `smtp_password`: SMTP password
- `smtp_auth_file`: Path to the SMTP auth file (default: /opt/elastalert/smtp_auth.yaml)
- `target_email`: Email address to receive alerts

You can override these in your playbook or inventory as needed.

---

SMTP Configuration
------------------
To enable email alerts, configure the following variables (example for Gmail):

```yaml
smtp_host: "smtp.gmail.com"
smtp_port: 587
smtp_user: "your_email@gmail.com"
smtp_password: "your_app_password"
target_email: "recipient@example.com"
```

The role will generate an SMTP authentication file at the path specified by `smtp_auth_file` (default: `/opt/elastalert/smtp_auth.yaml`):

```yaml
user: {{ smtp_user }}
password: {{ smtp_password }}
```

**Note:** For Gmail, you may need to use an App Password and enable "Less secure app access" or use OAuth2.

---

Example Playbook
----------------
```yaml
- hosts: elastalert_servers
  roles:
    - role: elastalert
      smtp_user: "your_email@gmail.com"
      smtp_password: "your_app_password"
      target_email: "recipient@example.com"
      elasticsearch_host: "elasticsearch.local"
      elasticsearch_port: 9200
```

---

Detection Rules Included
-----------------------
This role provides three example ElastAlert rules in `roles/elastalert/templates/`:

1. **UserAPI Brute Force Login Attempt Detected** (`login_failures_rule.yaml.j2`)
   - **Purpose:** Detects 20 failed login attempts (HTTP 401) to `/api/auth/login` from the same IP within 3 minutes.
   - **How it works:**
     - Looks for logs in the `userapi-logs-*` index.
     - Filters for status_code 401 and request_path `/api/auth/login`.
     - Groups by `client_ip`.
     - Triggers an email alert if the threshold is exceeded.

2. **API Request Flooding Detected** (`ddos_rule.yaml.j2`)
   - **Purpose:** Detects potential DDoS or API abuse by flagging 1000+ requests from the same IP in 1 minute.
   - **How it works:**
     - Looks for logs in the `werkzeug-logs-*` index.
     - Groups by `client_ip`.
     - Triggers an email alert if the threshold is exceeded.

3. **5xx Server Error Detected** (`server_error_rule.yaml.j2`)
   - **Purpose:** Detects 10 or more HTTP 5xx errors from the same index in 5 minutes.
   - **How it works:**
     - Looks for logs in the `werkzeug-logs-*` index.
     - Filters for status_code between 500 and 599.
     - Triggers an email alert if the threshold is exceeded.

All rules are configured to use the SMTP settings and send alerts to the email specified by `target_email`.

---

Dependencies
------------
None outside of the requirements listed above.

---

License
-------
MIT-0

