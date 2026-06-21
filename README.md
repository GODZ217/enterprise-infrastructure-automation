# Enterprise Infrastructure Automation

Infrastructure automation with Red Hat Satellite managing 1000+ RHEL VMs, automated provisioning and hardening, Ansible-based configuration management, and Grafana monitoring dashboards for platform observability.

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              Enterprise Infrastructure Automation              │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Red Hat Satellite                        │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │    │
│  │  │   Satellite  │  │   Capsule    │  │  Capsule   │ │    │
│  │  │   Server     │  │  Server DC1  │  │  Server DC2│ │    │
│  │  │              │  │              │  │            │ │    │
│  │  │  ┌────────┐  │  │  ┌────────┐  │  │ ┌────────┐ │ │    │
│  │  │  │Content │  │  │  │ RHEL   │  │  │ │ RHEL   │ │ │    │
│  │  │  │ Views  │  │  │  │ 500+   │  │  │ │ 500+   │ │ │    │
│  │  │  └────────┘  │  │  │ VMs    │  │  │ │ VMs    │ │ │    │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Automation Layer                        │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │    │
│  │  │   Ansible    │  │Provisioning  │  │  Hardening │ │    │
│  │  │   Tower/AWX  │  │   Scripts    │  │  Scripts   │ │    │
│  │  │              │  │              │  │            │ │    │
│  │  │ • Playbooks  │  │ • Kickstart  │  │ • CIS      │ │    │
│  │  │ • Templates  │  │ • PXE Boot   │  │ • STIG     │ │    │
│  │  │ • Workflows  │  │ • Image Gen  │  │ • Bench    │ │    │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Infrastructure Layer                    │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │    │
│  │  │   VMware     │  │     RHEL     │  │ OpenShift  │ │    │
│  │  │   vSphere    │  │     VMs      │  │  Cluster   │ │    │
│  │  │              │  │              │  │            │ │    │
│  │  │ • Clusters   │  │ • App VMs   │  │ • Nodes    │ │    │
│  │  │ • Datastores │  │ • DB VMs    │  │ • Infra    │ │    │
│  │  │ • Resource   │  │ • Util VMs  │  │ • Worker   │ │    │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Monitoring & Observability              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │    │
│  │  │   Prometheus │  │   Grafana    │  │  Alert     │ │    │
│  │  │   Metrics    │  │  Dashboards  │  │  Manager   │ │    │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

## Features

- **Satellite Management**: Content views, errata, lifecycle environments
- **Patch Automation**: Automated security patching for 1000+ VMs
- **Provisioning**: Automated host provisioning with Kickstart
- **Hardening**: CIS benchmark and STIG compliance automation
- **Monitoring**: Custom Grafana dashboards for infrastructure metrics
- **Inventory**: Comprehensive CMDB with Satellite

## Prerequisites

- Red Hat Satellite 6.x subscription
- RHEL 8/9 systems
- Ansible Tower/AWX (optional)
- VMware vSphere environment

## Satellite Setup

### Server Installation

```bash
# Prerequisites
subscription-manager register \
  --org="Organization" \
  --activationkey="satellite-key"

dnf install satellite

# Run installer
satellite-installer \
  --scenario satellite \
  --foreman-admin-password <password> \
  --foreman-initial-organization "ExampleOrg" \
  --foreman-initial-location "DataCenter1"
```

### Capsule Server

```bash
# Register capsule
subscription-manager register \
  --org="ExampleOrg" \
  --activationkey="capsule-key"

dnf install satellite-capsule

# Configure capsule
satellite-installer \
  --scenario capsule \
  --foreman-proxy-content-parent-fqdn satellite.example.com \
  --foreman-proxy-register-in-foreman true \
  --foreman-proxy-foreman-base-url https://satellite.example.com \
  --foreman-proxy-trusted-hosts satellite.example.com \
  --foreman-proxy-trusted-hosts capsule-dc1.example.com
```

## Content Management

### Lifecycle Environments

```bash
# Create lifecycle environments
hammer lifecycle-environment create \
  --organization "ExampleOrg" \
  --name "Development" \
  --prior "Library"

hammer lifecycle-environment create \
  --organization "ExampleOrg" \
  --name "Testing" \
  --prior "Development"

hammer lifecycle-environment create \
  --organization "ExampleOrg" \
  --name "Production" \
  --prior "Testing"
```

### Content Views

```bash
# Create content view
hammer content-view create \
  --organization "ExampleOrg" \
  --name "RHEL8-Base" \
  --label "rhel8-base" \
  --description "Base RHEL 8 packages"

# Add repositories
hammer content-view add-repository \
  --organization "ExampleOrg" \
  --name "RHEL8-Base" \
  --repository "Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)"

# Publish version
hammer content-view publish \
  --organization "ExampleOrg" \
  --name "RHEL8-Base" \
  --description "Version 1.0 - Initial"

# Promote to lifecycle
hammer content-view version promote \
  --organization "ExampleOrg" \
  --content-view "RHEL8-Base" \
  --version 1.0 \
  --to-lifecycle-environment "Development"
```

## Host Management

### Register Hosts

```bash
# Register to Satellite with activation key
subscription-manager register \
  --org="ExampleOrg" \
  --activationkey="rhel8-prod-key" \
  --server-url=satellite.example.com

# Install katello-agent
dnf install katello-agent

# Apply errata
dnf update --security
```

### Provisioning via Hammer CLI

```bash
hammer host create \
  --name "web-server-01" \
  --organization "ExampleOrg" \
  --location "DataCenter1" \
  --hostgroup "RHEL8-Prod-Web" \
  --compute-resource "vCenter" \
  --compute-attributes "{\
    \"cpus\": 4, \
    \"memory_mb\": 8192, \
    \"cluster\": \"Production-Cluster\", \
    \"path\": \"/Datacenters/DC1/vm/\", \
    \"guest_id\": \"rhel8_64Guest\", \
    \"scsi_controller_type\": \"VirtualLsiLogicController\" \
  }"
```

## Ansible Automation

### Patch Management Playbook

```yaml
---
- name: Patch Management Automation
  hosts: all
  become: yes
  
  tasks:
    - name: Check available updates
      yum:
        list: updates
      register: available_updates
    
    - name: Apply security updates
      yum:
        name: '*'
        security: yes
        state: latest
      when: available_updates.results | length > 0
    
    - name: Reboot if needed
      reboot:
        reboot_timeout: 600
      when:
        - ansible_facts.packages['kernel'] is defined
        - ansible_facts.packages['kernel'][0].version != ansible_facts.kernel
```

### Hardening Playbook (CIS Benchmark)

```yaml
---
- name: RHEL CIS Hardening
  hosts: all
  become: yes
  
  tasks:
    - name: Disable unused filesystems
      modprobe:
        name: "{{ item }}"
        state: absent
      loop:
        - cramfs
        - freevxfs
        - jffs2
        - hfs
        - hfsplus
        - squashfs
        - udf
    
    - name: Set password policies
      lineinfile:
        path: /etc/security/pwquality.conf
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: '^minlen', line: 'minlen = 14' }
        - { regexp: '^minclass', line: 'minclass = 4' }
        - { regexp: '^maxrepeat', line: 'maxrepeat = 3' }
    
    - name: Configure auditd
      lineinfile:
        path: /etc/audit/auditd.conf
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: '^max_log_file', line: 'max_log_file = 100' }
        - { regexp: '^space_left_action', line: 'space_left_action = email' }
        - { regexp: '^admin_space_left_action', line: 'admin_space_left_action = halt' }
```

## Monitoring

### Prometheus Node Exporter

```bash
# Install node exporter on all hosts
dnf install prometheus-node-exporter
systemctl enable --now prometheus-node-exporter

# Verify metrics
curl http://localhost:9100/metrics
```

### Grafana Dashboard Queries

```promql
# CPU usage per host
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk space
(node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_free_bytes{mountpoint="/"})
  / node_filesystem_size_bytes{mountpoint="/"} * 100

# Patch compliance
# (requires Satellite Prometheus exporter)
satellite_host_errata_count{severity="security"}
```

## Commands

| Command | Description |
|---------|-------------|
| `hammer host list` | List all managed hosts |
| `hammer host update` | Update host attributes |
| `hammer erratum list` | List available errata |
| `hammer content-view publish` | Publish content view |
| `ansible-playbook -i inventory patch.yml` | Run patch playbook |
| `govc vm.info <name>` | VMware VM details |
| `oscap xccdf eval --profile cis` | CIS benchmark scan |

## Best Practices

- Use content views with lifecycle environments
- Implement phased patching (Dev → Test → Prod)
- Automate host registration with activation keys
- Standardize host groups for consistent configuration
- Regular compliance scanning and reporting
- Monitor patching compliance metrics
- Document all automation workflows

## Security Considerations

- Use CIS benchmarks for OS hardening
- Implement STIG compliance for regulated environments
- Regular vulnerability scanning integration
- Secure Satellite communication with TLS
- RBAC for Satellite and Ansible Tower
- Audit logging for all administrative actions
- Regular backup of Satellite configuration
- Patch critical vulnerabilities within SLA timelines
