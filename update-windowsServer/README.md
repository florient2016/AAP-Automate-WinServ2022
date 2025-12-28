# update-windowsServer — Windows Update playbook

This folder contains an Ansible playbook to search for, apply, and manage Windows Updates on Windows Server hosts using the `ansible.windows.win_updates` module.

Files
- `Win-Update.yaml` — Playbook that searches for updates, logs results, applies updates, and reboots if necessary.

Prerequisites
- Ansible control node with the `ansible.windows` collection installed:

```bash
ansible-galaxy collection install ansible.windows
```

- Python WinRM support on the control node or in your Execution Environment:

```bash
pip install pywinrm requests
```

- A target Windows host with WinRM configured and reachable from the control node.

Playbook overview
- The playbook runs `ansible.windows.win_updates` to search for available updates by category (Security, Critical, Updates).
- It writes search results to `C:\Windows\Temp\check_update.log` and a detailed log to `C:\Windows\Temp\windows_update.log`.
- It applies updates (category `Updates`) and uses `ansible.windows.win_reboot` to reboot the host when required.

Variables (in `Win-Update.yaml`)
- `log_file`: Path for detailed update log (default: `C:\Windows\Temp\windows_update.log`).
- `check_update_file`: Path for a short JSON summary of available updates (default: `C:\Windows\Temp\check_update.log`).
- `KB`: Example KB number (e.g. `KB5010475`) — can be used for accept lists if adjusted in the playbook.

Inventory example
Create `inventory.ini` with WinRM connection settings:

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

Run the playbook
Using `ansible-playbook` from the control node:

```bash
ansible-playbook -i inventory.ini Win-Update.yaml
```

Using `ansible-navigator` with an Execution Environment image (optional):

```bash
ansible-navigator run Win-Update.yaml -i inventory.ini --execution-environment-image localhost:5000/win-chocolatey-ee:latest
```

Notes & tips
- The playbook sets `gather_facts: false` — if you rely on gathered facts, enable it by removing or changing that setting.
- To limit updates to a specific KB, uncomment and set `accept_list` in the `apply updates` task and pass `KB` variable.
- Logs are saved under `C:\Windows\Temp` — adjust paths if needed for your environment.

Troubleshooting
- WinRM connection failures: verify listeners, firewall rules, and credentials.
- No updates found: ensure the target can reach Microsoft Update services and proxy settings are correct.
- Reboot not occurring: check `update_result['reboot_required']` in task results; manual reboot may be needed in some cases.

License
- Public domain for personal/educational use (modify as required).

---
Created for the `Win-Update.yaml` playbook.

