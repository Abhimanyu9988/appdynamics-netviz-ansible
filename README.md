# 🚀 AppDynamics NetViz Agent Ansible Deployment

![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)
![AppDynamics](https://img.shields.io/badge/AppDynamics-0078D4?style=for-the-badge&logo=appdynamics&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

Deploy AppDynamics Network Visibility (NetViz) Agent automatically using Ansible with OAuth authentication. This solution extends the excellent AppDynamics automation ecosystem to include NetViz agents with secure, automated deployment.

## ⚡ Quick Start

```bash
# 1. Clone repository
git clone https://github.com/Abhimanyu9988/appdynamics-netviz-ansible.git
cd appdynamics-netviz-ansible

# 2. Configure your credentials
cp vars/controller.yaml.example vars/controller.yaml
nano vars/controller.yaml

# 3. Create encrypted vault for credentials
ansible-vault create vars/vault.yaml

# 4. Deploy NetViz
ansible-playbook -i inventory/localhosts.yml deploy-netviz.yaml --ask-vault-pass
```

Check your AppDynamics Controller → Infrastructure → Network Visibility for results.

## 📋 Prerequisites

- **Ubuntu 18.04+** with sudo access
- **Ansible 2.9+** installed
- **AppDynamics account** with NetViz license
- **Network connectivity** to AppDynamics SaaS

## 🔧 Configuration

### 1. Controller Settings (`vars/controller.yaml`)

```yaml
# AppDynamics Controller
controller_host: "your-tenant.saas.appdynamics.com"
controller_port: "443"
controller_ssl_enabled: "true"
controller_account_name: "your-account-name"
controller_access_key: "{{ vault_controller_access_key }}"

# NetViz Configuration
netviz_version: "25.7.0.3267"
netviz_webservice_port: 3892
netviz_max_memory: "512MB"
netviz_log_level: "info"
```

### 2. Encrypted Credentials (`vars/vault.yaml`)

Create with `ansible-vault create vars/vault.yaml`:

```yaml
# AppDynamics Credentials (encrypted)
vault_oauth_username: "your-email@domain.com"
vault_oauth_password: "your-password"
vault_controller_access_key: "your-controller-access-key"
```

### 3. Inventory Configuration

**Localhost deployment** (`inventory/localhost.yml`):
```yaml
[netviz_servers]
localhost ansible_connection=local ansible_become=yes
```

**Multi-host deployment** (`inventory/hosts.yml`):
```yaml
[netviz_servers]
server1 ansible_host=10.0.1.10 ansible_user=ubuntu
server2 ansible_host=10.0.1.11 ansible_user=ubuntu

[netviz_servers:vars]
ansible_become=yes
ansible_ssh_private_key_file=~/.ssh/your-key.pem
```

## 🚀 Deployment

### Localhost Deployment
```bash
ansible-playbook -i inventory/localhost.yml deploy-netviz.yml --ask-vault-pass
```

### Multi-Host Deployment
```bash
ansible-playbook -i inventory/hosts.yml deploy-netviz.yml --ask-vault-pass
```

## ✅ Verification

The playbook automatically verifies the deployment. You can also check manually:

```bash
# Service status
sudo systemctl status appd-netviz

# Network port
sudo netstat -tlnp | grep 3892

# Process status
sudo /opt/appdynamics/netviz/bin/appd-netviz.sh status

# Logs
sudo tail -f /opt/appdynamics/netviz/logs/agent.log
```

## 🔧 How It Works

### Authentication Flow
1. **OAuth Login**: Uses your AppDynamics credentials to get access token
2. **Secure Download**: Downloads NetViz DEB package with token authentication
3. **Checksum Verification**: Validates file integrity using SHA256
4. **Installation**: Installs and configures NetViz agent
5. **Service Management**: Starts and enables systemd service

### What Gets Installed
- **NetViz Agent**: Installed to `/opt/appdynamics/netviz`
- **SystemD Service**: `appd-netviz` service for management
- **Configuration**: Agent configured to connect to your controller
- **User Management**: `appdynamics` user with proper permissions

## 📁 Repository Structure

```
appdynamics-netviz-ansible/
├── deploy-netviz.yml              # Main deployment playbook
├── vars/
│   ├── controller.yaml.example    # Controller configuration template
│   └── vault.yaml.example         # Credentials template
├── inventory/
│   ├── localhost.yml              # Localhost inventory
│   └── hosts.yml.example          # Multi-host inventory template
├── templates/
│   └── agent_config.lua.j2        # NetViz configuration template
└── README.md                      # This file
```

## 🔧 Troubleshooting

### Authentication Issues
```bash
# Test your credentials manually
curl -X POST https://identity.msrv.saas.appdynamics.com/v2.0/oauth/token \
  -H "Content-Type: application/json" \
  -d '{"username":"your-email","password":"your-password","scopes":["download"]}'
```

### Service Issues
```bash
# Check service logs
sudo journalctl -u appd-netviz -f

# Check configuration
sudo cat /opt/appdynamics/netviz/conf/agent_config.lua

# Restart service
sudo systemctl restart appd-netviz
```

### Common Solutions

| Issue | Solution |
|-------|----------|
| Authentication fails | Verify credentials in vault.yaml |
| Download fails | Check network access to download.appdynamics.com |
| Service won't start | Check port 3892 availability |
| No metrics in controller | Verify controller connectivity and account settings |

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **AppDynamics** for the powerful Network Visibility platform
- **Ansible Community** for excellent automation tools

## 💬 Support

- **Issues**: [GitHub Issues](https://github.com/Abhimanyu9988/appdynamics-netviz-ansible/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Abhimanyu9988/appdynamics-netviz-ansible/discussions)

---

**Found this useful? ⭐ Please star this repository!**