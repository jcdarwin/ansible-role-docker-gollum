mebooks.ansible-role-docker-gollum
==================================

Installs [Gollum](https://github.com/gollum/) using the [mebooks/docker-gollum-alpine docker image](https://hub.docker.com/r/mebooks/docker-gollum-alpine/).

We presume the use of [Traefik](traefik.io) as our reverse proxy, and use git to back our wiki content up to a repository.

We assume that we've probably run:

* [ansible-role-users](https://github.com/jcdarwin/ansible-role-users)
* [ansible-role-common](https://github.com/jcdarwin/ansible-role-common)
* [ansible-role-docker](https://github.com/jcdarwin/ansible-role-docker)

The above all chain as dependencies of our main dependency:

* [ansible-role-docker-traefik](https://github.com/jcdarwin/ansible-role-docker-traefik)

We mount the wiki content from: `/etc/gollum/content`

Requirements
------------

Developed and tested on *Ubuntu Server 16.10 Yakkety*, but should work on other OSes.

It's running on Docker.

Deploy Key
----------

`ansible-role-users` generates a single SSH keypair on the server at `/root/.ssh/id_rsa` and prints the public key — it does **not** register it with any git remote for you. Before this role can clone (and before the hourly cron job can push) `gollum.remote`, that public key must be manually added as a **deploy key with write access** on the remote repo:

* GitHub: repo → Settings → Deploy keys → Add deploy key → paste `cat /root/.ssh/id_rsa.pub`, tick "Allow write access"
* Bitbucket: repo → Repository settings → Access keys → Add key

If the remote is being changed on an already-provisioned server (e.g. migrating providers), also make sure the new host's key is trusted before the first clone/push, e.g. `ssh-keyscan -H github.com >> /root/.ssh/known_hosts` — the `git` task in `tasks/main.yml` sets `accept_hostkey: yes` so a fresh deploy handles this automatically, but it won't retroactively fix a checkout that already exists on disk.

Role Variables
--------------

- `gollum.domain` defaults to *domain.tld*
- `gollum.source`: defaults to *domain.tld*
- `gollum.owner` defaults to *admin*
- `gollum.owner_password_encrypted` defaults to *WHATEVER*
- `gollum.remote` defaults to *git@github.com:whoever/wiki.git*

Dependencies
------------

- `mebooks.ansible-role-docker-traefik`

Example Playbook
----------------

```yml
- hosts: remote
  remote_user: deploy
  become: true
  become_user: root
  become_method: sudo

  vars:
    traefik:
      traefik_testing: true
      owner: administrator
      domain: mebooks.co.nz
      email: mebooks.support@gmail.com
      users:
      # owner_password / owner_password_encrypted are defined in the unversioned group_vars/remote
      - username: "{{ owner }}"
        password: "{{ owner_password }}"
        acl:
        - traefik

    gollum:
      domain: wiki.mebooks.co.nz
      source: mebooks.co.nz
      owner: "{{ owner }}"
      owner_password_encrypted: "{{ owner_password_encrypted }}"
      remote: git@github.com:nzmebooks/wiki.git

  roles:
    # We presume we've already run ansible-role-users and ansible-role-common
    # The following role is listed as dependencies in ansible-role-docker-traefik/meta/main.yml:
    # - role: ansible-role-docker
    # The following role is listed as dependencies in ansible-role-docker-ghost/meta/main.yml:
    # - role: ansible-role-docker-traefik
    - role: ansible-role-docker-gollum
```

License
-------

GPLv3

Author Information
------------------

Jason Darwin
