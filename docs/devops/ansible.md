# Ansible

## What is Ansible?

**Ansible** is an open-source **Configuration Management** and **provisioning** tool by Red Hat. It lets you define the desired state of a system in YAML files and apply it automatically via SSH — no agent required on the target machine.

Key properties:

- **Agentless** — connects over SSH, nothing to install on the target
- **Declarative** — you describe what the system should look like, not how to get there
- **Idempotent** — running the same playbook twice produces the same result

> ### 💡 **Why Ansible over a shell script?**
>
> A bash script runs top to bottom and breaks on partial failures. Ansible tracks state, skips already-done steps, and gives you structured output with per-task status.

| Skill | Score |
|-------|:-----:|
| Explain what Ansible is and when to use it | 5 |
| Describe agentless architecture | 4 |
| Explain idempotency in the context of Ansible | 4 |

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Inventory** | List of target hosts (can be a simple `.ini` file) |
| **Playbook** | YAML file that defines what to do and on which hosts |
| **Role** | Reusable unit grouping tasks, variables, and files by concern |
| **Task** | A single action (install a package, create a file, run a command) |
| **Module** | Built-in function used in tasks (`apt`, `copy`, `git`, `file`, etc.) |
| **Handler** | Task triggered only when notified (e.g. restart a service) |
| **Variable** | Dynamic value injected into tasks and templates |
| **Tag** | Label on a task or role to allow selective execution |

---

## File Structure

```
ansible/
├── playbook.yml          # Entry point — defines plays and roles
├── inventory.ini         # Target hosts
├── vars/
│   └── packages.yml      # Shared variables (package lists, versions)
└── roles/
    ├── base/tasks/main.yml      # System packages
    ├── dev/tasks/main.yml       # Dev tools (Go, Node, Python, gh)
    ├── devops/tasks/main.yml    # DevOps tools (Docker, kubectl, Terraform, AWS CLI)
    └── dotfiles/tasks/main.yml  # Clone repo and symlink config files
```

---

## Inventory

The inventory defines where Ansible connects. For a local machine:

```ini
[local]
localhost ansible_connection=local
```

---

## Playbook

A playbook contains one or more **plays**. Each play targets a group of hosts and applies a set of roles or tasks.

```yaml
- name: System packages and tools
  hosts: local
  become: true                        # Run as root (sudo)
  vars_files:
    - vars/packages.yml
  vars:
    real_user: "{{ ansible_env.SUDO_USER | default(ansible_env.USER) }}"
  roles:
    - { role: base,   tags: base }
    - { role: dev,    tags: dev }
    - { role: devops, tags: devops }

- name: User dotfiles
  hosts: local
  become: false                       # Run as the current user
  vars_files:
    - vars/packages.yml
  roles:
    - { role: dotfiles, tags: dotfiles }
```

- `become: true` escalates to root — equivalent to running with `sudo`
- Split into two plays to handle system-level vs user-level concerns separately
- `vars_files` loads shared variables from an external file

| Skill | Score |
|-------|:-----:|
| Write a basic playbook | 5 |
| Use `become` for privilege escalation | 4 |
| Split plays by privilege level | 3 |

---

## Roles

A **role** organises tasks by concern. The entry point is always `roles/<name>/tasks/main.yml`.

### apt packages

```yaml
# roles/base/tasks/main.yml
- name: Update apt cache
  ansible.builtin.apt:
    update_cache: true
    cache_valid_time: 3600

- name: Install base packages
  ansible.builtin.apt:
    name: "{{ apt_base_packages }}"
    state: present
```

### Binary install (idempotent)

For tools not available via apt (Go, AWS CLI), check existence before downloading:

```yaml
- name: Check if Go is installed
  ansible.builtin.stat:
    path: /usr/local/go/bin/go
  register: go_binary

- name: Download Go tarball
  ansible.builtin.get_url:
    url: "https://go.dev/dl/go{{ go_version }}.linux-amd64.tar.gz"
    dest: /tmp/go.tar.gz
    mode: "0644"
  when: not go_binary.stat.exists
```

- `register` captures task output into a variable
- `when` conditionally skips a task based on that output

### Dotfiles — clone and symlink

```yaml
- name: Clone dotfiles repository
  ansible.builtin.git:
    repo: https://github.com/LionelPinheiroDuarte/dotfiles.git
    dest: "{{ ansible_env.HOME }}/repos/github/dotfiles"
    update: false
  when: not dotfiles_dir.stat.exists

- name: Symlink dotfiles to $HOME
  ansible.builtin.file:
    src: "{{ ansible_env.HOME }}/repos/github/dotfiles/{{ item.src }}"
    dest: "{{ ansible_env.HOME }}/{{ item.dest }}"
    state: link
    force: true
  loop:
    - { src: "bash/.bashrc",    dest: ".bashrc" }
    - { src: "vim/.vimrc",      dest: ".vimrc" }
    - { src: "tmux/.tmux.conf", dest: ".tmux.conf" }
    - { src: ".gitconfig",      dest: ".gitconfig" }
```

- `loop` iterates a task over a list of items
- `update: false` prevents Ansible from pulling changes on re-runs

| Skill | Score |
|-------|:-----:|
| Structure a project with roles | 5 |
| Use `register` and `when` for conditional tasks | 4 |
| Install tools from binary tarballs | 3 |
| Use `loop` to iterate tasks | 4 |

---

## Variables

Variables are centralised in `vars/packages.yml` and shared across plays via `vars_files`.

```yaml
apt_base_packages:
  - vim
  - git
  - tmux
  - curl
  - bat

apt_dev_packages:
  - python3
  - python3-pip
  - python3-venv

go_version: "1.23.4"
node_major_version: "22"
kubectl_version: "v1.31"
```

Used in tasks with Jinja2 syntax:

```yaml
name: "{{ apt_base_packages }}"
url: "https://go.dev/dl/go{{ go_version }}.linux-amd64.tar.gz"
```

| Skill | Score |
|-------|:-----:|
| Define and use variables | 5 |
| Use `vars_files` to share variables across plays | 4 |

---

## Third-party apt Repositories

For tools like Docker, kubectl, or Terraform, the pattern is always:

1. Download and dearmor the GPG key
2. Add the apt repository
3. Install the package

```yaml
- name: Download and dearmor Docker GPG key
  ansible.builtin.shell: |
    curl -fsSL https://download.docker.com/linux/{{ ansible_distribution | lower }}/gpg \
      | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  args:
    creates: /etc/apt/keyrings/docker.gpg      # skip if file already exists

- name: Add Docker apt repository
  ansible.builtin.apt_repository:
    repo: "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] ..."
    state: present
    filename: docker

- name: Install Docker
  ansible.builtin.apt:
    name: [docker-ce, docker-ce-cli, containerd.io]
    state: present
    update_cache: true
```

- `args: creates:` makes a `shell` task idempotent — it skips if the file exists
- `ansible_distribution` is a built-in fact (`Ubuntu`, `Debian`, etc.)

---

## Core Workflow

```bash
# Run the full playbook (prompts for sudo password)
ansible-playbook playbook.yml -i inventory.ini -K

# Run only specific roles via tags
ansible-playbook playbook.yml -i inventory.ini -K --tags dev
ansible-playbook playbook.yml -i inventory.ini -K --tags dotfiles

# Dry run — show what would change without applying
ansible-playbook playbook.yml -i inventory.ini --check

# Verbose output
ansible-playbook playbook.yml -i inventory.ini -K -v
```

| Skill | Score |
|-------|:-----:|
| Run a playbook with `ansible-playbook` | 5 |
| Use `--tags` to target specific roles | 4 |
| Use `--check` for dry runs | 3 |

---

## Testing with Docker

A Dockerfile lets you test the playbook against a clean Ubuntu image without touching your real machine:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y ansible-core git sudo curl

RUN useradd -m -s /bin/bash testuser \
    && echo "testuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER testuser
WORKDIR /home/testuser

COPY --chown=testuser:testuser . /home/testuser/repos/github/dotfiles

CMD ["ansible-playbook", \
     "/home/testuser/repos/github/dotfiles/ansible/playbook.yml", \
     "-i", "/home/testuser/repos/github/dotfiles/ansible/inventory.ini"]
```

```bash
# Build and run
docker build -t dotfiles-test -f ansible/Dockerfile .
docker run --rm dotfiles-test
```

- Run from the repo root so `COPY . ...` includes all files
- `NOPASSWD:ALL` avoids the `-K` password prompt inside the container
- `--rm` cleans up the container after the run

| Skill | Score |
|-------|:-----:|
| Test Ansible playbooks in Docker | 4 |

---

### 🔑 **Important Keywords:**
`playbook`, `role`, `task`, `module`, `inventory`, `become`, `idempotent`, `agentless`, `vars_files`, `register`, `when`, `loop`, `tags`, `ansible-playbook`, `apt`, `handlers`
