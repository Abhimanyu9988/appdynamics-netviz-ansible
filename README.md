# 🚀 AppDynamics NetViz Agent Ansible Deployment

![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)
![AppDynamics](https://img.shields.io/badge/AppDynamics-0078D4?style=for-the-badge&logo=appdynamics&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)

Deploy AppDynamics Network Visibility (NetViz) Agent automatically using Ansible with OAuth authentication on **Linux** and **Windows**. This solution extends the excellent AppDynamics automation ecosystem to include NetViz agents with secure, automated deployment.

## ⚡ Quick Start

### 🐧 Linux Deployment
```bash
# 1. Clone repository
git clone https://github.com/Abhimanyu9988/appdynamics-netviz-ansible.git
cd appdynamics-netviz-ansible

# 2. Configure credentials
cp vars/controller.yaml.example vars/controller.yaml
ansible-vault create vars/vault.yaml

# 3. Deploy NetViz on Linux
ansible-playbook -i inventory/localhosts.yml deploy-netviz.yaml --ask-vault-pass
```

### 🪟 Windows Deployment
```bash
# 1. Setup WinRM on target Windows machines first
# 2. Configure Windows inventory
cp inventory/windows-hosts.yml.example inventory/windows-hosts.yml

# 3. Deploy NetViz on Windows
ansible-playbook -i inventory/windows-hosts.yml deploy-netviz-windows.yml --ask-vault-pass
```

## 📋 Prerequisites

### 🐧 Linux Requirements
- **Ubuntu 18.04+** with sudo access
- **Ansible 2.9+** installed
- **Network connectivity** to AppDynamics SaaS

### 🪟 Windows Requirements  
- **Windows Server 2016+** or **Windows 10+**
- **PowerShell 3.0+** installed
- **WinRM configured** for Ansible connectivity
- **Administrator access** on target machines

### 🔑 Common Requirements
- **AppDynamics account** with NetViz license
- **OAuth credentials** for downloading agents

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
netviz_log_level: "info"
```

### 2. Encrypted Credentials (`vars/vault.yaml`)

Create with `ansible-vault create vars/vault.yaml`:

```yaml
# AppDynamics OAuth Credentials
vault_oauth_username: "your-email@domain.com"
vault_oauth_password: "your-password"
vault_controller_access_key: "your-controller-access-key"
```

### 3. Platform-Specific Inventory

#### 🐧 Linux Inventory (`inventory/localhost.yml`)
```yaml
[netviz_servers]
localhost ansible_connection=local ansible_become=yes
```

#### 🪟 Windows Inventory (`inventory/windows-hosts.yml`)
```yaml
[windows_netviz_servers]
windows-server-1 ansible_host=10.0.1.100 ansible_user=Administrator

[windows_netviz_servers:vars]
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_port=5985
```

## 🚀 Deployment

### 🐧 Linux Deployment

#### Localhost
```bash
ansible-playbook -i inventory/localhost.yml deploy-netviz.yml --ask-vault-pass
```

#### Multiple Linux Hosts
```bash
ansible-playbook -i inventory/hosts.yml deploy-netviz.yml --ask-vault-pass
```

### 🪟 Windows Deployment

#### Single Windows Host
```bash
ansible-playbook -i inventory/windows-localhost.yml deploy-netviz-windows.yml --ask-vault-pass
```

#### Multiple Windows Hosts
```bash
ansible-playbook -i inventory/windows-hosts.yml deploy-netviz-windows.yml --ask-vault-pass
```

## 🔧 Windows Setup (One-time)

### Configure WinRM on Windows Targets

Run this PowerShell script as Administrator on each Windows target:

```powershell
# Enable WinRM
Enable-PSRemoting -Force

# Configure WinRM for Ansible
winrm quickconfig -q
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
winrm set winrm/config '@{MaxTimeoutms="1800000"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'

# Configure firewall
netsh advfirewall firewall add rule name="WinRM-HTTP" dir=in localport=5985 protocol=TCP action=allow
```

## ✅ Verification

### 🐧 Linux Verification
```bash
# Service status
sudo systemctl status appd-netviz

# Network port
sudo netstat -tlnp | grep 3892

# Process status
sudo /opt/appdynamics/netviz/bin/appd-netviz.sh status
```

### 🪟 Windows Verification
```powershell
# Service status
Get-Service "AppDynamics NetViz"

# Network port
Test-NetConnection -Port 3892

# Process status
Get-Process | Where-Object {$_.ProcessName -like "*netviz*"}

# Or use the verification script
.\scripts\verify-windows.ps1
```

## 🔧 How It Works

### Authentication Flow (Both Platforms)
1. **OAuth Login**: Uses your AppDynamics credentials to get access token
2. **Secure Download**: Downloads appropriate package (DEB for Linux, ZIP for Windows)
3. **Checksum Verification**: Validates file integrity using SHA256
4. **Platform-Specific Installation**: 
   - **Linux**: DEB package installation + systemd service
   - **Windows**: ZIP extraction + Windows service registration
5. **Service Management**: Starts and configures the appropriate service

### Platform Differences

| Feature | Linux | Windows |
|---------|-------|---------|
| **Package Type** | DEB (3.8 MB) | ZIP (27.4 MB) |
| **Installation Path** | `/opt/appdynamics/netviz` | `C:\AppDynamics\NetViz` |
| **Service Manager** | systemd | Windows Service Manager |
| **User Account** | `appd-netviz` | Local System |
| **Configuration** | Lua config file | Lua config file |
| **Dependencies** | `curl`, `wget`, `tcpdump` | None required |

## 📁 Repository Structure

```
appdynamics-netviz-ansible/
├── deploy-netviz.yml              # Linux deployment playbook
├── deploy-netviz-windows.yml      # Windows deployment playbook
├── vars/
│   ├── controller.yaml.example    # Controller configuration template
│   └── vault.yaml.example         # Credentials template
├── inventory/
│   ├── localhost.yml              # Linux localhost
│   ├── hosts.yml.example          # Linux multi-host
│   ├── windows-localhost.yml      # Windows localhost
│   └── windows-hosts.yml.example  # Windows multi-host
├── templates/
│   └── agent_config.lua.j2        # NetViz configuration template
├── scripts/
│   └── verify-windows.ps1         # Windows verification script
└── README.md                      # This file
```

## 🔧 Troubleshooting

### 🐧 Linux Issues

| Issue | Solution |
|-------|----------|
| Permission denied | `sudo chown -R appd-netviz:appd-netviz /opt/appdynamics/netviz` |
| Service won't start | Check port 3892: `sudo netstat -tlnp \| grep 3892` |
| Authentication fails | Verify credentials in vault.yaml |

### 🪟 Windows Issues

| Issue | Solution |
|-------|----------|
| WinRM connection failed | Run WinRM setup script as Administrator |
| Service won't start | Check Windows Event Logs for NetViz errors |
| Download fails | Verify network access to download.appdynamics.com |

### 🔧 Common Solutions

#### Test OAuth Authentication
```bash
curl -X POST https://identity.msrv.saas.appdynamics.com/v2.0/oauth/token \
  -H "Content-Type: application/json" \
  -d '{"username":"your-email","password":"your-password","scopes":["download"]}'
```

#### Check Controller Connectivity
```bash
# Linux
telnet your-controller.saas.appdynamics.com 443

# Windows
Test-NetConnection your-controller.saas.appdynamics.com -Port 443
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Test on both Linux and Windows
4. Submit a pull request

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

**Platform Support**: 🐧 Linux (Ubuntu/Debian) | 🪟 Windows (Server 2016+/Windows 10+)