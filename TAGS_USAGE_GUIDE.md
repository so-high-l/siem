# ELK Stack Tags Usage Guide

This guide explains how to use Ansible tags to run specific components of the ELK stack independently.

## 🏷️ Available Tags

### **Component Tags** (Run individual services)
- `elasticsearch` - Deploy only Elasticsearch
- `kibana` - Deploy only Kibana
- `logstash` - Deploy only Logstash
- `beats` - Deploy only Filebeat and Metricbeat
- `elastalert` - Deploy only ElastAlert2

### **Group Tags** (Run multiple related services)
- `elk` - Run all ELK stack components (Elasticsearch, Logstash, Kibana, Beats, ElastAlert)
- `setup` - Run only system setup tasks (dependencies, limits, etc.)
- `verification` - Run only verification tasks (service checks, status display)

### **Special Tags**
- `always` - These tasks always run regardless of other tags

## 🚀 Usage Examples

### **Run Complete ELK Stack**
```bash
# Run everything (default behavior)
ansible-playbook playbook.yml --ask-vault-pass

# Or explicitly run all ELK components
ansible-playbook playbook.yml --ask-vault-pass --tags elk
```

### **Run Individual Components**

#### **Deploy Only Elasticsearch**
```bash
# Run Elasticsearch with system setup
ansible-playbook playbook.yml --ask-vault-pass --tags "setup,elasticsearch"

# Run only Elasticsearch (if system is already set up)
ansible-playbook playbook.yml --ask-vault-pass --tags elasticsearch
```

#### **Deploy Only Kibana**
```bash
# Run Kibana with system setup
ansible-playbook playbook.yml --ask-vault-pass --tags "setup,kibana"

# Run only Kibana (if system is already set up)
ansible-playbook playbook.yml --ask-vault-pass --tags kibana
```

#### **Deploy Only Logstash**
```bash
# Run Logstash with system setup
ansible-playbook playbook.yml --ask-vault-pass --tags "setup,logstash"

# Run only Logstash (if system is already set up)
ansible-playbook playbook.yml --ask-vault-pass --tags logstash
```

#### **Deploy Only Beats**
```bash
# Run Beats with system setup
ansible-playbook playbook.yml --ask-vault-pass --tags "setup,beats"

# Run only Beats (if system is already set up)
ansible-playbook playbook.yml --ask-vault-pass --tags beats
```

#### **Deploy Only ElastAlert**
```bash
# Run ElastAlert with system setup
ansible-playbook playbook.yml --ask-vault-pass --tags "setup,elastalert"

# Run only ElastAlert (if system is already set up)
ansible-playbook playbook.yml --ask-vault-pass --tags elastalert
```

### **Run System Setup Only**
```bash
# Run only system setup tasks (dependencies, limits, etc.)
ansible-playbook playbook.yml --ask-vault-pass --tags setup
```

### **Run Verification Only**
```bash
# Run only verification tasks (service checks, status)
ansible-playbook playbook.yml --ask-vault-pass --tags verification
```

## 🔄 Common Use Cases

### **1. Initial Setup**
```bash
# First time setup - run system preparation
ansible-playbook playbook.yml --ask-vault-pass --tags setup
```

### **2. Deploy Core ELK Stack**
```bash
# Deploy Elasticsearch, Logstash, and Kibana
ansible-playbook playbook.yml --ask-vault-pass --tags "elasticsearch,logstash,kibana"
```

### **3. Add Monitoring Later**
```bash
# Add Beats for monitoring after core stack is running
ansible-playbook playbook.yml --ask-vault-pass --tags beats
```

### **4. Add Alerting Later**
```bash
# Add ElastAlert for alerting after core stack is running
ansible-playbook playbook.yml --ask-vault-pass --tags elastalert
```

### **5. Update Specific Component**
```bash
# Update only Kibana configuration
ansible-playbook playbook.yml --ask-vault-pass --tags kibana
```

## ⚠️ Important Notes

### **Dependencies**
- **Elasticsearch** should be deployed first as other components depend on it
- **Kibana** and **Logstash** require Elasticsearch to be running
- **Beats** require Logstash to be running
- **ElastAlert** requires Elasticsearch to be running

### **Recommended Deployment Order**
1. `setup` - System preparation
2. `elasticsearch` - Core search engine
3. `logstash` - Log processing
4. `kibana` - Web interface
5. `beats` - Data collection
6. `elastalert` - Alerting

### **Skip Tags**
You can also use `--skip-tags` to exclude specific components:

```bash
# Run everything except ElastAlert
ansible-playbook playbook.yml --ask-vault-pass --skip-tags elastalert

# Run everything except verification
ansible-playbook playbook.yml --ask-vault-pass --skip-tags verification
```

## 🔧 Troubleshooting with Tags

### **Check What Will Run**
```bash
# Dry run to see what tasks will execute
ansible-playbook playbook.yml --ask-vault-pass --tags elasticsearch --check

# List all tags in the playbook
ansible-playbook playbook.yml --list-tags
```

### **Debug Specific Component**
```bash
# Run with verbose output for debugging
ansible-playbook playbook.yml --ask-vault-pass --tags kibana -vvv
```

### **Re-run Failed Tasks**
```bash
# If a specific component failed, re-run just that component
ansible-playbook playbook.yml --ask-vault-pass --tags elasticsearch
```

## 📋 Tag Reference Table

| Tag | Description | Dependencies |
|-----|-------------|--------------|
| `setup` | System preparation (dependencies, limits) | None |
| `elasticsearch` | Elasticsearch installation and configuration | `setup` |
| `logstash` | Logstash installation and configuration | `setup`, `elasticsearch` |
| `kibana` | Kibana installation and configuration | `setup`, `elasticsearch` |
| `beats` | Filebeat and Metricbeat installation | `setup`, `logstash` |
| `elastalert` | ElastAlert2 installation and configuration | `setup`, `elasticsearch` |
| `elk` | All ELK components | `setup` |
| `verification` | Service verification and status checks | All components |
| `always` | Tasks that always run | None |

## 🎯 Quick Commands Reference

```bash
# Complete deployment
ansible-playbook playbook.yml --ask-vault-pass

# Individual components
ansible-playbook playbook.yml --ask-vault-pass --tags elasticsearch
ansible-playbook playbook.yml --ask-vault-pass --tags kibana
ansible-playbook playbook.yml --ask-vault-pass --tags logstash
ansible-playbook playbook.yml --ask-vault-pass --tags beats
ansible-playbook playbook.yml --ask-vault-pass --tags elastalert

# System operations
ansible-playbook playbook.yml --ask-vault-pass --tags setup
ansible-playbook playbook.yml --ask-vault-pass --tags verification

# Multiple components
ansible-playbook playbook.yml --ask-vault-pass --tags "elasticsearch,kibana"
ansible-playbook playbook.yml --ask-vault-pass --tags "setup,elasticsearch,logstash"
``` 