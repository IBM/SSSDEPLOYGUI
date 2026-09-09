# SSSDEPLOYGUI
IBM Spectrum Scale System — Deploy from GUI Container Manager

Hosts the **Deploy from GUI web UI container** on an IBM Utility BareMetal node (5149-23E).
The Deploy from GUI container provides a browser-based interface for deploying and upgrading IBM Storage Scale System (ESS) nodes. It communicates with the SSS API Server container (SSSAPI) to execute deployment operations.

> **Note:** `SSSRCL`, `SSSAPI`, and `SSSDEPLOYGUI` have been unified into the [SSSMANAGER](https://github.ibm.com/IBMSpectrumScale/SSSMANAGER) repository. Use `SSSMANAGER` for all new deployments. This repository is maintained for standalone use only.

---

## Quick start

```bash
./startDeployGUIContainer
```

This is the **only script you need to run**. It auto-detects system settings, writes a sanitised `deployguimgr.yml`, and starts the container.

> **Prerequisite:** The SSS API Server (SSSAPI) must already be running before you start the Deploy from GUI container.

---

## Usage

```text
./startDeployGUIContainer [options]

Options:
  -v, --verbose              Display verbose messages in the console.
  -f, --file-name FILE       Restore the container from a local tar file.
                             (default: SSSDEPLOYGUI.tar)
  -p, --port PORT            Port on which the Deploy from GUI server listens.
                             (default: 30443)
  -ap, --api-port PORT       Port on which the API server listens.
                             (default: 46443)
  -i, --campus-interface IF  Name of the interface connected to the campus
                             network. (default: campus)
  -vn, --image-version VER   Deploy GUI image version to deploy.
                             (default: 7.0.2.0)
```

### Examples

```bash
# Start using default settings (pulls image from IBM registry)
./startDeployGUIContainer

# Start from a local tarball
./startDeployGUIContainer -f SSSDEPLOYGUI.tar

# Override the GUI listen port
./startDeployGUIContainer -p 30443

# Override both the GUI port and the API port it connects to
./startDeployGUIContainer -p 30443 -ap 46443

# Override the image version
./startDeployGUIContainer -vn 7.0.1.1

# Verbose output
./startDeployGUIContainer -v
```

---

## Prerequisites

### OS and software

- RHEL / Red Hat compatible Linux
- `podman` installed and accessible
- `nmcli` installed and accessible
- User-level systemd lingering enabled (`loginctl enable-linger`)
- SSS API Server container running on the same host (see [SSSAPI](https://github.ibm.com/IBMSpectrumScale/SSSAPI))

### Network interfaces

| Interface | Purpose |
|-----------|---------|
| `campus`  | Public network — connects the Utility host to the Internet |
| `virbr1`  | RAS bridge — internal communication with EMSVM |

### OS user

Create the `deployguiadmin` user before running the script:

```bash
sudo useradd deployguiadmin
sudo passwd deployguiadmin
```

No additional sudoers entry is required for `deployguiadmin`.

### Hosts file

Replace `/etc/hosts` with the provided template (or append its contents):

```bash
sudo cp utilityBareMetal_hosts /etc/hosts
```

Ensure the RAS IP for EMSVM (`10.23.16.2 emsvm`) is correct for your system.

### IBM Container Registry login

```bash
podman login cp.icr.io --username cp --password <API_Key>
```

The Deploy from GUI image is pulled from:

```
preprod.icr.io/cp/scalesystem/sss_deploygui
```

---

## Configuration

`deployguimgr.yml` is written automatically by `startDeployGUIContainer`. You can also edit it manually for advanced settings:

```yaml
CONTAINER:
    CONTAINER_HOSTNAME: utilityBareMetal-deploygui-official
    CONTAINER_DOMAIN_NAME: gpfs.local

    UTILITY_HOSTNAME: utilityBareMetal

    CAMPUS_INTERFACE: campus
    CAMPUS_INTERFACE_IP: 9.114.12.77   # ← auto-detected at runtime

    RAS_INTERFACE: virbr1
    RAS_INTERFACE_IP: 10.23.16.1

    DEPLOY_GUI_PORT: 30443             # HTTPS web UI port
    API_PORT: 46443                    # API Server port this container connects to
    SSH_PORT: 30022                    # SSH port (development only)

    IMAGE_NAME: preprod.icr.io/cp/scalesystem/sss_deploygui
    IMAGE_VERSION: 7.0.2.0

    LOG:  /home/deployguiadmin/log
    BKUP: /home/deployguiadmin/backup
```

> `CAMPUS_INTERFACE_IP` is auto-detected from the campus interface at runtime. The value in the YAML is used as a fallback only.

---

## Port reference

| Host port | Container port | Purpose |
|-----------|----------------|---------|
| 30443     | 443            | HTTPS web UI |
| 46443     | —              | Outbound connection to API container |
| 30022     | 22             | SSH (development only) |

---

## Systemd service

After a successful start the container is registered as a user-level systemd service that auto-starts on login:

```bash
# Check status
systemctl --user status container-utilityBareMetal-deploygui-official.service

# Start / stop
systemctl --user start  container-utilityBareMetal-deploygui-official.service
systemctl --user stop   container-utilityBareMetal-deploygui-official.service
```

---

## Logs

Local execution logs are written to `./logs/` and are also copied to `/var/log/deploygui/startDeployGUIContainer/` if that directory exists.

---

## Project structure

```text
SSSDEPLOYGUI/
├── startDeployGUIContainer  ← entry point (run this)
├── deployguimgr.yml         ← configuration file (auto-generated / editable)
├── deployguimgr             ← legacy low-level manager (deprecated, use startDeployGUIContainer)
├── utilityBareMeral_hosts   ← /etc/hosts template for the Utility node
└── classes/
    └── deployguimgr_yml.py  ← config + container lifecycle class
```

---

## Related repositories

| Repository | Purpose |
|------------|---------|
| [SSSMANAGER](https://github.ibm.com/IBMSpectrumScale/SSSMANAGER) | Unified manager for RCL + API + Deploy GUI |
| [SSSRCL](https://github.ibm.com/IBMSpectrumScale/SSSRCL) | RCL container manager |
| [SSSAPI](https://github.ibm.com/IBMSpectrumScale/SSSAPI) | SSS API Server container manager |
