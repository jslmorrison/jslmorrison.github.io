+++
title = "Ansistrano"
date = 2023-05-21
tags = ["ansible", "devops"]
categories = ["devops", "ansible"]
draft = false
+++

An excellent tool for deploying and rolling back project deployments.

<!--more-->

> Ansistrano is built on top of Ansible to leverage all its power to deploy applications.

This can be installed in host via `ansible-galaxy` as follows:

```bash
ansible-galaxy install ansistrano.deploy ansistrano.rollback
```

An example playbook to deploy project to production server:

```yaml
---
- name: Deploy to production webserver
  hosts: production
  become: true

  vars_files:
    - ./ansible/vars/deploy_vault.yaml
    - ./ansible/vars/deploy_vars.yaml

  vars:
    release_console_path: "{{ ansistrano_release_path.stdout }}/bin/console"
    release_var_path: "{{ ansistrano_release_path.stdout }}/var"
    release_log_path: "{{ ansistrano_shared_path }}/var/log"
    ansistrano_deploy_via: git
    ansistrano_deploy_to: /var/www/html
    ansistrano_keep_releases: 3
    ansistrano_git_repo: ${GITHUB_REPO}
    ansistrano_git_branch: master
    ansistrano_git_depth: 1
    ansistrano_git_identity_key_remote_path: /root/.ssh/id_rsa
    # hooks
    ansistrano_after_symlink_shared_tasks_file: "{{ playbook_dir }}/after-symlink-shared.yaml"

  environment:
    APP_ENV: prod

  roles:
    - { role: ansistrano.deploy, tags: deploy }
```
