# Настройка ansible

Устанавливаем ansible и sshpass для подключения к хостам по логину/паролю:

```bash
apt install ansible sshpass
```

В `/etc/ansible/ansible.cfg` пишем

```ini
inventory = /etc/ansible/inventory
host_key_checking = False
```

В `/etc/ansible/inventory` пишем

```ini
[all:vars]
ansible_ssh_user=sshuser
ansible_ssh_pass=P@ssw0rd
ansible_python_interpreter=python3

[servers]
srv-hq   ansible_ssh_private_key_file=/root/.ssh/id_rsa ansible_port=2023
srv-br   ansible_ssh_private_key_file=/root/.ssh/id_rsa ansible_port=2023
pve1-br
pve2-br
cicd-hq
worker1-hq
worker2-hq
k8s-master-hq
k8s-worker1-hq
k8s-worker2-hq

[clients]
cli-hq
cli-br

[networking]
sw-hq
sw-br
rtr-hq   ansible_host=10.0.10.1 ansible_user=admin ansible_connection=network_cli ansible_network_os=esr
rtr-br   ansible_host=10.0.20.1 ansible_user=admin ansible_connection=network_cli ansible_network_os=esr
```

Пишем плейбук

```yaml
---
- hosts: servers,clients,sw-hq,sw-rtr
  tasks:
  - name: Collect info from srv, cli, sw
    lineinfile:
      path: "/etc/ansible/output.yml"
      line: "{{ ansible_facts['fqdn'] }} - {{ ansible_facts['default_ipv4']['address'] }}"
      create: yes
    delegate_to: localhost

- hosts: rtr-hq,rtr-br
  tasks:
  - name: Collect info from rtr-hq and rtr-br
    esr_command:
      commands: show run | i hostname
    register: cmd_output

  - name: Write info to file
    lineinfile:
      path: "/etc/ansible/output.yml"
      line: "{{ cmd_output.stdout[0].split()[1] }} - {{ ansible_host }}"
      create: yes
    delegate_to: localhost
```
