# Ansible Example for KernelCare

## Objective
Use Ansible + ePortal API to apply Kernelcare patch to multiple servers with minimal manual interaction.
Gradually deploy patchset to multiple feeds `staging` followed by `production`.

### Example Configuration:
```
ePortal
    |
    |---+ staging
    |   |
    |   +--- kce-test-c7-01
    |   |
    |   +--- kce-test-c7-02
    |
    |---+ production
        |
        +--- kce-prod-c7-01
        |
        +--- kce-prod-c7-02
```

### Workflow:
* Create two feeds `staging` and `production`
* Install Kernelcare agent to all servers
* Register two servers `kce-test-c7-01/02` with the `staging` feed
* Register two servers `kce-prod-c7-01/02` with the `production` feed
* Deploy the latest Kernelcare patchset to the `staging` feed
* Make sure the deployed patchset works fine on the `staging` servers
* Deploy the latest Kernelcare patchset to the `production` feed


## References
* ePortal API (https://docs.tuxcare.com/eportal-api/)
* kcarectl CLI (https://docs.tuxcare.com/live-patching-services/#command-line-tools)
* Deployment Automation (https://docs.tuxcare.com/eportal/#deployment-automation)

## Prerequisites
* ePortal Installation
* ePortal Patchsource Selection on Web UI
  * manual only, no API available but typically one-time setup
* Passwordless authentication on all servers
  * SSH public key authentication is setup
  * Passwordless sudo is enabled
  * otherwise enable 'ask_pass' and 'ask_sudo_pass' in ansible.cfg
* Ansible packages
  * ansible-core
  * community.general
  * jmespath

```bash
# Ansible installation (E.g. Ubuntu 22.04)
sudo add-apt-repository ppa:ansible/ansible
sudo apt install ansible-core -y
ansible-galaxy collection install community.general
pip install jmespath
```

## Configurations
### Server SSH Login User
Specify the username to ssh login to target servers.

```bash
$ cat ~/.ansible.cfg
[defaults]
remote_user=admin
```

### Target Servers to be patched
Add your servers ip address or hostname.
```bash
$ cat inventory/servers
[kernelcares:children]
staging
production

[staging]
kce-test-c7-01
kce-test-c7-02

[production]
kce-prod-c7-01
kce-prod-c7-02
```

### ePortal Access Credentials
```bash
$ cat group_vars/all/all.yml
eportal_api:  http://<eportal ip address>
eportal_user: <eportal account name>
eportal_pass: <eportal account password>
eportal_feeds:
  - staging
  - production
```

## Usage

### 1. Setup
Run the playbook on the ePortal server.

```bash
ansible-playbook -i inventory kcare.yml
```

* Create two feeds `staging` and `production` on ePortal
* Install Kernelcare agent to all servers
* Register two servers `kce-test-c7-01/02` with the `staging` feed
* Register two servers `kce-prod-c7-01/02` with the `production` feed

### 2. Deploy the latest Patchset to the `staging` servers
```bash
ansible-playbook -i inventory kcare.yml --skip-tags setup --extra-vars "feed=staging patchset=latest"
```

* Fetch and enable the latest patchset for the `staging` feed
* Apply the patchset to the `staging` servers

### 3. Any test on the staging servers before rolling out to the production servers

### 4. Deploy the latest Patchset to the `production` servers
```bash
ansible-playbook -i inventory kcare.yml --skip-tags setup --extra-vars "feed=production patchset=latest"
```

* Fetch and enable the latest patchsets for the `production` feed
* Apply the patchset to the `production` servers

### 5. (optional) Rollback to an certain patchset when a problem occurs
```bash
ansible-playbook -i inventory kcare.yml --skip-tags setup --extra-vars "feed=production patchset=K20231215_08 rollback=yes"
```

* Rollback the patchset for the `production` feed to `K20231215_08`

## To-Do List (Limitations)
* Better idempotency
  * Skip installation of kernelcare agent if it's already installed
  * Skip feed creation if it's already available
  * Skip rollout of patchset if nothing changes
