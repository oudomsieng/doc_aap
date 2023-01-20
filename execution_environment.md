Environnement d'Exécution
=========================

# Prérequis

## Podman

```
# dnf install ansible-builder
```

# Fichier de definition d'ansible-builder

```yaml
---
version: 2

ansible_config: 'ansible.cfg'

dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt
additional_build_steps:
  prepend: |
    RUN pip3 install --upgrade
  append:
    - RUN echo Un command de test post-install

images:
  base_image:
    name: URL_VERS_AUTOMATIONHUB/BASE_PODMAN_IMAGE:VERSION
  builder_image:
    name: URL_VERS_AUTOMATIONHUB/BUILDER_PODMAN_IMAGE:VERSION
    
```

`ansible.cfg`: permet d'inclure un fichier de configuration d'Ansible et la configuration galaxy et token pour récuperer les collections et les images de Automation Hub Privé

```ini
[galaxy]
server_list = private_automation_hub

[galaxy_server.private_automation_hub]
url=URL_VERS_AUTOMATIONHUB
token=TOKEN_AUTOMATIONHUB
```

`dependencies`: permet de définir les fichier de définition des collections galaxy, des librairies python et des packages systèmes à installer dans l'environnement d'exécution.

Le fichier ``requirements.yml`` pour les collections Ansible à inclure dans l'environnement d'exécution

```yaml
---

collections:
  - name: vmware.vmware_rest
    version: 2.2.0
  - name: community.efficientip
    version: 1.0.3
```

Le fichier ``requirements.txt`` pour les modules Python à inclure dans l'environnement d'exécution

```
SOLIDserverRest>=2.3.2
aiohttp
```

Le fichier ``bindep.txt`` pour les packages systèmes à installer:

```
git [platform:rpm]
```

# Démarrage de build

```
$ ansible-builder build --tag vmware-ee-image --container-runtime podman
```

# Valider l'image de l'environnement d'exécution

Si vous n'avez pas déjà installé Ansible Runner, vous pouvez vous référer à [Ansible Runner Doc](https://ansible-runner.readthedocs.io/en/stable/) pour obtenir des conseils. Vous trouverez ci-dessous un exemple de playbook (nous l'appellerons test.yml) qui peut être exécuté via Ansible Runner pour s'assurer que l'environnement d'exécution fonctionne :

```yaml
---

- hosts: localhost
  connection: local
  tasks:
    - name: collect a list of the datacenters
      vmware.vmware_rest.vcenter_datacenter_info:
      register: my_datacenters

    - name: Build a list of all the clusters
      vmware.vmware_rest.vcenter_cluster_info:
      register: all_the_clusters

```

## Lancer le test

```
$ ansible-runner playbook --container-image=vmware-ee-image test.yml
```

# Publier l'image

```
$ podman login -u=username -p=password URL_VERS_AUTOMATIONHUB
$ podman push URL_VERS_AUTOMATIONHUB/vmware-ee-image
```
