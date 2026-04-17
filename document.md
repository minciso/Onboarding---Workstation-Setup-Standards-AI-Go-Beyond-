# Ansible Onboarding — Setup Document

## Repository Tree

```
ansible-onboarding/
├── .gitattributes
├── .gitignore
├── .pre-commit-config.yaml        # yamllint pre-commit hook
├── .vscode/
│   ├── .editorconfig              # editor whitespace/indent rules
│   └── settings.json             # VS Code Ansible + Python venv config
├── .venv/                         # isolated Python virtual environment (not committed)
├── git-sign-test/                 # sandbox repo for testing SSH commit signing
│   └── .pre-commit-config.yaml
├── ansible.cfg                    # project-level Ansible configuration
├── requirements.txt               # pinned pip dependencies
└── README.md                      # environment setup guide
```

---

## What Makes This Setup Team-Friendly

Using a **pinned `requirements.txt` with a project-local `.venv`** makes onboarding reproducible for every team member.
Anyone cloning the repo can run:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

and immediately get the exact same versions of `ansible-core`, `ansible-lint`, `yamllint`, and every transitive dependency — no version drift, no "works on my machine" surprises.
The `.pre-commit-config.yaml` then enforces consistent YAML style across all contributors automatically on every commit.

---

## Pitfall Avoided — Global pip Install

Installing Ansible globally via `pip install ansible` would have polluted the system Python environment and made it impossible to run different Ansible versions across projects.
By using a **virtual environment (`.venv`)** scoped to this repo:

- The system Python stays clean.
- Different projects can pin different `ansible-core` versions without conflict.
- The `.gitignore` excludes `.venv/` so the environment is never accidentally committed.

A second pitfall to be aware of: **the SSH agent is not automatically started in WSL sessions**.
Running `ssh-add -l` may return `Could not open a connection to your authentication agent` — meaning Ansible cannot authenticate to remote hosts even if your key exists on disk.

To fix this, start the agent and load your key before running any playbook:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l   # verify key fingerprint is listed
```

Consider adding the `eval` line to your `~/.bashrc` so it starts automatically in every WSL session.

---

## Proxy and CA Considerations (Corporate Setup)

This project was set up in a personal environment, so no corporate proxy or custom CA certificate configuration was required.
In a corporate network, the following steps are typically needed:

**1. Set proxy for pip installs:**
```bash
pip install --proxy http://proxy.corp.example.com:8080 ansible ansible-lint
# or set environment variables:
export HTTP_PROXY=http://proxy.corp.example.com:8080
export HTTPS_PROXY=http://proxy.corp.example.com:8080
```

**2. Trust corporate CA certificates for pip:**

If pip fails with SSL certificate errors, import the corporate CA bundle:

```bash
pip install --cert /path/to/corporate-ca-bundle.crt ansible
# or configure permanently:
pip config set global.cert /path/to/corporate-ca-bundle.crt
```

**3. Trust the CA inside WSL:**

```bash
sudo cp corporate-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

**4. Ansible connections through a bastion/jump host:**

Add a jump host to `ansible.cfg` or `~/.ssh/config` if managed hosts are behind a corporate firewall:

```ini
# ansible.cfg
[ssh_connection]
ssh_args = -o ProxyJump=bastion.corp.example.com -o ControlMaster=auto -o ControlPersist=60s
```

Since none of these were needed here, no proxy or CA config is present in the current `ansible.cfg`.
