+++
title = "Ansible Vault"
date = 2023-12-29
tags = ["ansible", "devops"]
draft = false
+++

How to use [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) to keep sensitive data in your playbooks hidden and protected.

<!--more-->

> Ansible Vault is a feature of Ansible that allows users to encrypt sensitive data such as passwords, API keys, and other confidential information.


## Vault file {#vault-file}

Define sensitive key value pairs for usage in your playbook, in a dedicated yaml file, (e.g. `vars/vault.yaml`) for encryption using `ansible-vault` tool.

```yaml
# vars/vault.yaml
---
vault_my_database_root_password: rootPassword123
vault_my_secret_api_key: api-key-123
```

This file can be encrypted using the `ansible-vault` tool in the terminal as follows:

```bash
ansible-vault encrypt vars/vault.yaml
```

The contents of the file will now be encrypted and when viewed will look something like:

```txt
$ANSIBLE_VAULT;1.1;AES256
63643633356133633136643833316532363065346461326330643634306564646138306334613961
6333323134333432663831623737376338613233623239310a303461666537313064663130373738
36396362363163656163373061353234353966306533663131633433643132643466396264663636
6634353932626435650a613561626133616139386166376539633633343939393162386535306464
38323336336536376166353835636531616435356132323030666566393565613065333666336463
33333434343531613764333336353965666430636431303863613464373935363565306237393666
62663638396364383034343331613361656235373164396363346338663335366538656236343435
35643839656630616631
```

You will be asked to provide a vault password at this stage, be sure to remember it as it will be required anytime you run a playbook that requires access to the encrypted variables.
To view the unencrypted contents use:

```bash
ansible-vault view vars/vault.yaml
```

If you want to change your vault password, you can do so by running:

```bash
ansible-vault rekey var/vault.yaml
```


## Playbook {#playbook}

To make use of the encrypted variables in your playbook, follow these steps:

-   create a new vars file to set new key value pairs:

<!--listend-->

```yaml
# vars/vars.yaml
---
my_database_root_password: "{{ vault_my_database_root_password }}"
my_secret_api_key: "{{ vault_my_secret_api_key  }}"
```

-   reference these files in playbook:

<!--listend-->

```yaml
# playbook.yaml
---
- name: Configure server

  vars_files:
    - ./vars/vault.yaml
    - ./vars/vars.yaml

    tasks:
      - name: debug the vault vars
        ansible.builtin.debug:
          msg: "rootPassword: {{ my_database_root_password }} apiKey: {{ my_secret_api_key }}"
        tags:
          - debug
```

When the playbook is ran the unencrypted values will be shown in output.

```bash
ansible-playbook -i hosts.ini -t debug --ask-vault-password playbook.yaml
```

output contains:

```bash
"msg": "rootPassword: rootPassword123 apiKey: api-key-123"
```

Providing the vault password each time is a bit of a nuisance, so you can store the password in a file, the content of which should only contain the actual vault password. Then use a modified command to run the playbook:

```bash
ansible-playbook -i hosts.ini -t debug --vault-password-file vaultpswd playbook.yaml
```

Obviously, don't commit that file.
