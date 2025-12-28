# Windows Chocolatey Execution Environment

This directory contains the Ansible Automation Platform (AAP) Execution Environment (EE) configuration for managing Windows servers with Chocolatey package manager.

## Overview

This Execution Environment is built on Red Hat Enterprise Linux 9 minimal image and includes:
- **Python Dependencies**: `requests`, `pywinrm`, and `boto3` for Windows PowerShell remote management and AWS integration
- **Ansible Collections**: `chocolatey.chocolatey` for Chocolatey package management on Windows

## Prerequisites

Before building this Execution Environment, ensure you have:

- **ansible-builder** installed (version 3.0 or later)
  ```bash
  pip install ansible-builder
  ```
- **Docker** or **Podman** installed and running
- Access to Red Hat registry or Docker Hub for base images

## Directory Structure

```
chocolatey-ee/
├── win-chocolate-ee.yaml    # Execution Environment definition
├── requirements.txt          # Python package requirements
├── requirements.yml          # Ansible collection requirements
└── README.md                 # This file
```

## Building the Execution Environment

### Step 1: Build with ansible-builder

login to your ansible automation platform repository
```bash
podman login <AAP repos links> # e.g: podman login myregistry.example.com
```

Run the following command from this directory to build the Execution Environment image:

```bash
mkdir dir-chocolatey
cd dir-chocolatey
ansible-builder build -t win-chocolatey-ee:latest -f win-chocolatey.yaml
```

**Explanation of the command:**
- `ansible-builder build`: Initiates the build process
- `-t win-chocolatey-ee:latest`: Sets the image tag (name and version)
- `.`: Uses the current directory (where `win-chocolate-ee.yaml` is located)

### Expected Build Output

The build process will:
1. Create a temporary Containerfile from the EE definition
2. Build the base image with all dependencies
3. Install Python requirements from `requirements.txt`
4. Install Ansible collections from `requirements.yml`
5. Execute custom build steps (prepend and append steps)
6. Generate the final image

Once complete, you'll see output confirming the image build was successful:
```
Successfully built win-chocolatey-ee:latest
```

## Managing Image Tags

### View Built Images

To see your newly built image:

```bash
docker images | grep win-chocolatey-ee
```

### Create Additional Tags

If you need to create multiple tags for versioning or different registries:

```bash
# Tag with version number
docker tag win-chocolatey-ee:latest win-chocolatey-ee:1.0

# Tag for local registry
docker tag win-chocolatey-ee:latest localhost:5000/win-chocolatey-ee:latest

# Tag with registry prefix
docker tag win-chocolatey-ee:latest myregistry.example.com/win-chocolatey-ee:latest
```

## Pushing to Local Repository

### Option 1: Local Docker Registry (Recommended)

If you have a local Docker registry running on port 5000:

```bash
# Tag the image for your local registry
docker tag win-chocolatey-ee:latest localhost:5000/win-chocolatey-ee:latest

# Push to local registry
docker push localhost:5000/win-chocolatey-ee:latest

# Verify the push
docker pull localhost:5000/win-chocolatey-ee:latest
```

### Option 2: Set up a Local Registry (if not already running)

```bash
# Run a local Docker registry container
docker run -d -p 5000:5000 --restart=always --name local-registry registry:2

# Tag and push to it
docker tag win-chocolatey-ee:latest localhost:5000/win-chocolatey-ee:latest
docker push localhost:5000/win-chocolatey-ee:latest
```

### Option 3: Save and Load Locally

For environments without a running registry:

```bash
# Save the image to a tar file
docker save win-chocolatey-ee:latest -o win-chocolatey-ee.tar

# Transfer the tar file to another system (if needed)

# Load the image on another system
docker load -i win-chocolatey-ee.tar
```

## Using the Execution Environment

Once the image is built, you can use it in your Ansible Automation Platform instance:

```yaml
# In your AAP controller configuration
execution_environments:
  - name: Windows Chocolatey EE
    image: win-chocolatey-ee:latest
```

Or use it directly with ansible-navigator:

```bash
ansible-navigator run playbook.yml --execution-environment-image localhost:5000/win-chocolatey-ee:latest
```

## Complete Build and Push Workflow

```bash
# 1. Build the image
ansible-builder build -t win-chocolatey-ee:latest .

# 2. Tag for local registry
docker tag win-chocolatey-ee:latest localhost:5000/win-chocolatey-ee:latest

# 3. Push to local registry
docker push localhost:5000/win-chocolatey-ee:latest

# 4. Verify the image is available
docker pull localhost:5000/win-chocolatey-ee:latest
```

## Troubleshooting

### Build Fails with "Base Image Not Found"

Ensure you have internet access or pull the base image first:
```bash
docker pull registry.redhat.io/ansible-automation-platform-24/ee-minimal-rhel9:latest
```

### Registry Connection Issues

Check if your local registry is running:
```bash
docker ps | grep registry

# Or restart it
docker restart local-registry
```

### Permission Denied Errors

On Linux systems, you may need to use `sudo`:
```bash
sudo docker push localhost:5000/win-chocolatey-ee:latest
```

## Dependencies Reference

### Python Packages
- **requests** (>=2.31.0): HTTP library for Python
- **pywinrm** (>=0.4.3): Python library for WinRM (Windows Remote Management)
- **boto3**: AWS SDK for Python (for AWS operations)

### Ansible Collections
- **chocolatey.chocolatey**: Official Ansible collection for managing Chocolatey packages on Windows

## Additional Resources

- [Ansible Builder Documentation](https://ansible-builder.readthedocs.io/)
- [Execution Environments Guide](https://ansible-automation-platform.readthedocs.io/en/latest/reference/execution-environments/)
- [Chocolatey Collection](https://galaxy.ansible.com/ui/repo/published/chocolatey/chocolatey/)
- [Docker Registry Documentation](https://docs.docker.com/registry/)

## Notes

- The base image is Red Hat Enterprise Linux 9 minimal, ensuring a lightweight and secure foundation
- Custom build steps are included to support Windows server management scenarios
- Ensure the Python requirements file is named `requirements.txt` (not `rrequirements.txt`)
- The execution environment definition uses version 3 format for Ansible Builder
