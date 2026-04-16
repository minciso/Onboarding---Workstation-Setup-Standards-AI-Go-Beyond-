# Ansible Onboarding

A local Ansible environment for learning and practicing infrastructure automation with linting and validation tools.

## Environment Setup

### OS & Python Version

- **OS**: Windows 11 with WSL2 (Ubuntu)
- **Python**: 3.12.x
- **WSL Distro**: Ubuntu (accessed via `wsl` command)

### Installation Method

Ansible and related tools were installed using pip in a Python virtual environment:

```bash
# Create and activate venv
python3 -m venv .venv
source .venv/bin/activate  # On WSL/Linux/macOS
# .venv\Scripts\activate  # On Windows PowerShell

# Upgrade pip
python -m pip install --upgrade pip

# Install Ansible and linting tools
python -m pip install ansible ansible-lint yamllint

# Save dependencies for reproducibility
python -m pip freeze > requirements.txt
```

All dependencies are tracked in `requirements.txt` and can be reinstalled with:

```bash
pip install -r requirements.txt
```

**Key Packages**:

- `ansible==13.5.0` - Automation framework
- `ansible-core==2.20.4` - Core engine
- `ansible-lint==26.4.0` - Playbook linting
- `yamllint==1.38.0` - YAML validation
- `black==26.3.1` - Code formatting (ansible-lint dependency)

## VS Code Extensions

The following extensions are recommended and configured:

### 1. **Red Hat Ansible** (redhat.ansible)

- **Purpose**: Ansible playbook syntax highlighting, completion, and validation
- **Config**: `.vscode/settings.json`
  - `ansibleLint.enabled = true` - Enables real-time linting
  - `ansible.python.interpreterPath` - Points to venv Python interpreter

### 2. **Python** (ms-python.python)

- **Purpose**: Python language support and debugging
- **Config**:
  - `python.defaultInterpreterPath` - Uses venv interpreter
  - Integrates with ansible extension

### 3. **YAML** (redhat.vscode-yaml)

- **Purpose**: YAML syntax validation and completion
- **Config**:
  - `yaml.validate = true` - Enables YAML schema validation
  - Works alongside yamllint CLI tool

### Additional Settings

- `files.trimTrailingWhitespace = true` - Clean up trailing whitespace
- `editor.formatOnSave = true` - Auto-format on save

## SSH Configuration

### SSH Key Location

SSH keys should be stored in your local SSH config directory (not in this repo):

**Linux/macOS/WSL**:

```bash
~/.ssh/
├── id_rsa           # Private key (DO NOT COMMIT)
├── id_rsa.pub       # Public key
└── config           # SSH client config (optional)
```

**Windows**:

```
C:\Users\<username>\.ssh\
├── id_rsa
├── id_rsa.pub
└── config
```

### Ansible SSH Configuration

SSH connection parameters are defined in `ansible.cfg`:

```ini
[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

This enables:

- **Connection pipelining** - Reduces round trips to managed hosts
- **Control Master** - Reuses SSH connections for efficiency
- **Control Persist** - Keeps connections alive for 60 seconds

**Important**: Never commit private keys to the repository. Add to `.gitignore`:

```gitignore
.ssh/
*.pem
*.key
id_rsa
id_rsa.pub
```

## Git Configuration

### Git User Setup

#### View Current Git Configuration

Check your current git configuration at all levels:

```bash
# View repository-level config (most specific)
git config --local --list

# View global config (applies to all repos)
git config --global --list

# View system config (applies to all users on system)
git config --system --list

# View all configs combined
git config --list
```

Example output:

```
user.name=Your Name
user.email=your.email@example.com
core.editor=nano
...
```

#### Configure Git User Identity

**Local (repository-only)**:

```bash
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

**Global (all repositories)**:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Verify configuration**:

```bash
git config user.name    # Should show your configured name
git config user.email   # Should show your configured email
```

#### Configure Git Globally

Set up global git configuration:

```bash
# Set user name
git config --global user.name "Your Name"

# Set user email
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# View all global config
git config --global --list
```

#### SSH Key Setup

Locate and verify your SSH public key:

```bash
# List SSH keys
ls ~/.ssh

# Display your public key (for adding to remote servers)
cat ~/.ssh/id_ed25519.pub

# Set correct permissions (if needed)
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

#### Configure SSH Commit Signing (Optional)

Set up allowed signers for SSH-based commit verification:

```bash
# Create git config directory
mkdir -p ~/.config/git

# Edit allowed signers file
vi ~/.config/git/allowed_signers

# Add your SSH public key in format:
# your.email@example.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ...
```

Then configure git to use SSH signing:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

#### Test Signed Commits

Create a test repository to verify commit signing:

```bash
# Create test directory
mkdir git-sign-test
cd git-sign-test

# Initialize git repo
git init

# Configure signing (optional, if you set up SSH signing)
git config user.signingkey ~/.ssh/id_ed25519.pub

# Create a test commit
git commit --allow-empty -m "Test signed commit"

# View commit signature
git log --show-signature

# View commits with oneline format
git log --oneline
```

Example output of `git log --show-signature`:

```
commit ABC123DEF456... (HEAD -> master)
Good "git" signature for your.email@example.com with key SHA256:ABCDEF...
Author: Your Name <your.email@example.com>
Date:   Wed Apr 17 10:30:45 2026

    Test signed commit
```

## Virtual Environment & Running Tools

### Activating the Virtual Environment

**On WSL/Linux/macOS**:

```bash
source .venv/bin/activate
# Prompt will change to show (.venv)
```

**On Windows PowerShell**:

```powershell
.venv\Scripts\Activate.ps1
```

**On Windows Command Prompt**:

```cmd
.venv\Scripts\activate.bat
```

### Running Ansible Lint

Validate Ansible playbooks and roles:

```bash
# Lint all playbooks in current directory
ansible-lint

# Lint specific playbook
ansible-lint playbooks/main.yml

# Lint specific role
ansible-lint roles/webserver/

# Show detailed output
ansible-lint -v

# Generate report in JSON
ansible-lint -f json > lint-report.json
```

### Running yamllint

Validate YAML syntax and style:

```bash
# Lint all YAML files in current directory
yamllint .

# Lint specific file
yamllint playbooks/vars.yml

# Lint specific directory
yamllint inventories/

# Use custom config
yamllint -c .yamllint playbooks/
```

### Running Ansible

Execute playbooks:

```bash
# Run playbook
ansible-playbook playbooks/main.yml

# Run with inventory
ansible-playbook -i inventories/hosts.ini playbooks/main.yml

# Dry run (check mode)
ansible-playbook -i inventories/hosts.ini playbooks/main.yml --check

# Verbose output
ansible-playbook -i inventories/hosts.ini playbooks/main.yml -v
```

## Quick Start

1. **Setup environment** (first time only):

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

2. **Activate venv** (each session):

   ```bash
   source .venv/bin/activate
   ```

3. **Validate playbooks**:

   ```bash
   ansible-lint playbooks/
   yamllint .
   ```

4. **Run playbooks**:
   ```bash
   ansible-playbook playbooks/main.yml
   ```

## Troubleshooting

### Connection Issues

If you get SSH connection errors, check:

- SSH keys exist and have correct permissions (600 for private, 644 for public)
- SSH agent is running: `ssh-add -l`
- SSH config has correct HostName, User, and IdentityFile entries

### Interpreter Issues

If Ansible can't find Python on remote hosts:

- Check `ansible.cfg` setting: `interpreter_python = auto_silent`
- Manually specify: `ansible_python_interpreter: /usr/bin/python3` in inventory

### Lint Errors

- Run `ansible-lint -v` for detailed explanations
- Check `.ansible-lint` config file for rule customization
- Visit [ansible-lint rules](https://ansible.readthedocs.io/projects/lint/) for rule reference
