# Node Exporter Ansible Role

This Ansible role installs and configures the Prometheus Node Exporter on Linux systems. It is designed to be idempotent, meaning it can be run multiple times without changing the system state if the desired state is already achieved.

## Features

- **Idempotent installation**: Only installs if Node Exporter is not present or version differs
- **Force installation**: Override detection with `node_exporter__force_install` variable
- **Multi-architecture support**: Supports AMD64, ARM64, and ARM architectures
- **Systemd integration**: Creates and manages systemd service
- **Version management**: Installs specific versions of Node Exporter
- **Clean installation**: Downloads, extracts, installs, and cleans up temporary files

## Requirements

- **Target OS**: Linux (systemd-based distributions)
- **Ansible**: 2.9 or higher
- **Privileges**: Root access (uses `become: true`)
- **Network**: Internet access to download Node Exporter binary

## Supported Architectures

- `x86_64` / `amd64`
- `arm64` / `aarch64`
- `arm` / `armv7l`

## Role Variables

### Default Variables (`defaults/main.yml`)

| Variable | Default | Description |
|----------|---------|-------------|
| `node_exporter__version` | `"1.9.1"` | Version of Node Exporter to install |
| `node_exporter__force_install` | `false` | Force installation even if already installed |
| `node_exporter__supported_os` | `["linux"]` | List of supported operating systems |

### Computed Variables

| Variable | Description |
|----------|-------------|
| `node_exporter__os` | Detected operating system (lowercase) |
| `node_exporter__architecture` | Mapped architecture for download URL |
| `node_exporter__should_install` | Boolean determining if installation is needed |
| `node_exporter__systemd_installed` | Boolean checking if systemd is available |

## Dependencies

None.

## Example Playbook

### Basic Usage

```yaml
- hosts: monitoring
  become: true
  roles:
    - node_exporter
```

### Force Installation

```yaml
- hosts: monitoring
  become: true
  vars:
    node_exporter__force_install: true
  roles:
    - node_exporter
```

### Specific Version

```yaml
- hosts: monitoring
  become: true
  vars:
    node_exporter__version: "1.8.2"
  roles:
    - node_exporter
```

### Command Line Override

```bash
# Force reinstallation
ansible-playbook playbook.yml -e "node_exporter__force_install=true"

# Install specific version
ansible-playbook playbook.yml -e "node_exporter__version=1.8.2"
```

## Installation Process

1. **Detection**: Checks if Node Exporter service exists
2. **Download**: Downloads the appropriate binary for the target architecture
3. **Extract**: Extracts the binary from the archive
4. **Install**: Copies binary to system location and sets permissions
5. **Service**: Creates and enables systemd service
6. **Cleanup**: Removes temporary installation files

## File Structure

```
roles/node_exporter/
├── defaults/
│   └── main.yml          # Default variables
├── tasks/
│   ├── main.yml          # Main installation logic
│   └── linux.yml         # Linux-specific tasks
├── handlers/
│   └── main.yml          # Service restart handlers
├── templates/
│   └── node_exporter.service.j2  # Systemd service template
└── README.md             # This file
```

## Service Details

- **Service Name**: `node_exporter.service`
- **Default Port**: `9100`
- **User**: `node_exporter` (system user)
- **Binary Location**: `/usr/local/bin/node_exporter`
- **Auto-start**: Enabled on boot

## Testing

```bash
# Check if service is running
sudo systemctl status node_exporter

# Test metrics endpoint
curl http://localhost:9100/metrics

# Check process
ps aux | grep node_exporter
```

## Troubleshooting

### Service won't start

```bash
# Check service logs
sudo journalctl -u node_exporter -f

# Verify binary permissions
ls -la /usr/local/bin/node_exporter
```

### Architecture not supported

Check if your architecture is in the supported list and update `node_exporter__architecture_list` if needed.

### Download fails

Verify internet connectivity and that the specified version exists on GitHub releases.

## License

MIT

## Author Information

This role was created for monitoring infrastructure with Prometheus
