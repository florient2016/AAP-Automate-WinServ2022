# InstallPackage — Chocolatey playbook
#
# This folder contains an Ansible playbook to install Chocolatey and example packages
# on Windows Server 2022 using the `chocolatey.chocolatey` collection.

## Files

- `Install-chocolate.yaml` — Playbook to install Chocolatey and packages (putty, wireshark).

## Prerequisites

- Ansible (2.10+) installed on the control node.
- Python WinRM support: install `pywinrm` on the control node (or in your EE).
- The `chocolatey.chocolatey` Ansible collection installed.
- A reachable Windows host with WinRM configured to accept remote commands from the control node.

## Recommended setup commands

Install collection and Python requirements (adjust path if needed):

```bash
ansible-galaxy collection install -r requirements.yml
pip install -r ..\chocolatey-ee\rrequirements.txt
```

If you built an Execution Environment image (see `chocolatey-ee`), you can run the playbook inside the EE with `ansible-navigator`.

## Inventory example

Create a file `inventory.ini` with the Windows host and WinRM connection variables:

```ini
[windows]
winhost ansible_host=10.0.0.10

[windows:vars]
ansible_connection=winrm
ansible_user=Administrator
ansible_password=SecretPassword
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
```

Adjust `ansible_host`, `ansible_user`, and `ansible_password` for your environment.

## Run the playbook

Using `ansible-playbook` (control node):

```bash
ansible-playbook -i inventory.ini Install-chocolate.yaml
```

Using `ansible-navigator` with a local EE image (if available):

```bash
# example: use the EE image built as win-chocolatey-ee:latest
ansible-navigator run Install-chocolate.yaml -i inventory.ini --execution-environment-image localhost:5000/win-chocolatey-ee:latest
```

## Notes

- The playbook runs `chocolatey.chocolatey.win_chocolatey` tasks to install `chocolatey`, `putty`, and `wireshark`.
- If WinRM connectivity fails, verify firewall, WinRM listeners, and credentials on the target Windows host.
- The `rrequirements.txt` file in `../chocolatey-ee` contains Python deps (`pywinrm`, `requests`, `boto3`).

## Troubleshooting

- WinRM auth errors: confirm `ansible_winrm_transport` and credentials.
- Collection/module missing: run `ansible-galaxy collection install -r requirements.yml` from this folder.

---
Created for the `Install-chocolate.yaml` playbook.

