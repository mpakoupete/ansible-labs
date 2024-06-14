# Création des roles Ansible

## réinitialisation des hôtes

On veut repartir sur des hôtes vierges. On peut détruire à l'aide de vagrant.
Mais pour faire un néttoyage sommaire, nous allons supprimer les paquets nginx, apache2, mysql-server... installés précédemment.

Ecrire un playbook qui fait une boucle sur ces 3 paquets et les supprime de tous nos hôtes

<details><summary>Correction</summary>

```yaml
---
- name: Suppression des paquets
  hosts: all
  become: yes

  vars:
    paquet_a_supprimer:
      - nginx
      - apache
      - mysql-server

  tasks:
    - name: suppression de paquets
      apt:
        name: "{{ item }}"
        state: absent
      with_items: "{{ paquet_a_supprimer }}"
```

</details>

## Role Ansible

ansible-galaxy nous permet de télécharger des rôles et fournit également un excellent template par défaut pour créer nos propres rôles.

Nous allons répartir nos 3 serveurs en 3 groupes de serveurs : web, proxy, db; respectivement node1, node2, node3
Modifier le fichier inventaire en conséquence : 

<details><summary>Correction</summary>

```yaml
[web]
node1
[proxy]
node2
[db]
node3
```

</details>


### création de tâches d'installation de Apache2

Créer ces deux template suivants pour la configuration de notre serveur Apache2 et placez les dans un répertoire `templates` que vous allez créer

* Template 1 : `ports.conf.j2`
```
# /etc/apache2/sites-enabled/000-default.conf

Listen {{ http_port }}

<IfModule ssl_module>
        Listen {{ https_port }}
</IfModule>

<IfModule mod_gnutls.c>
        Listen {{ https_port }}
</IfModule>
```

* Template 2 : `index.html.j2`

```
<html>

<h1>{{ html_msg }}</h1>

</html>
```

Créez un playbook du nom de `installation_apache2.yml` qui :

* Install apache2 à la dernière version
* Place le template source `templates/ports.conf.j2` à la destination suivante `/etc/apache2/ports.conf`. Chaque fois que ce fichier est modifié, emettre une notification `redemarrer apache` (utiliser le plugin `notify`)
* Place le template `templates/index.html.j2` à la destination suivante `/var/www/html/index.html`. Chaque fois que ce fichier est modifié également, emettre une notification `redemarrer apache` (utiliser le plugin `notify`)
* Démarre le service `apache2`
* déclare ces 3 variables : `http_port: 8000`, `https_port: 4443`, `html_msg: "Hello Lab Orsys"`

<details><summary>Correction</summary>

```yaml
- hosts: web
  become: yes

  vars:
    http_port: 8000
    https_port: 4443
    html_msg: "Hello Lab Orsys"

  tasks:
    - name: installation apache2 dernière version
      apt: name=apache2 state=latest

    - name: ecrire fichier config apache2 ports.conf
      template:
        src: templates/ports.conf.j2
        dest: /etc/apache2/ports.conf
      notify: redemarrer apache

    - name: fichier index.html basic
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
      notify:
      - redemarrer apache

    - name: S'assuré que Apache est démarré
      service:
        name: apache2
        state: started
```

</details>

Modifiez le playbook précédent pour ajouter un handler. Ce handler devra réagir à chaque notification `redemarrer apache` pour redemarrer Apache2

<details><summary>Correction</summary>

```yaml
- hosts: web
  become: yes

  vars:
    http_port: 8000
    https_port: 4443
    html_msg: "Hello Lab Orsys"

  tasks:
    - name: installation apache2 dernière version
      apt: name=apache2 state=latest

    - name: ecrire fichier config apache2 ports.conf
      template:
        src: templates/ports.conf.j2
        dest: /etc/apache2/ports.conf
      notify: redemarrer apache

    - name: fichier index.html basic
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
      notify:
      - redemarrer apache

    - name: S'assuré que Apache est démarré
      service:
        name: apache2
        state: started
    
  handlers:
    - name: redemarrer apache
      service:
        name: apache2
        state: restarted
```

</details>


### Creation d'un role template

Un role n'est autre que la forme réutilisable de nos playbooks que nous allons décomposer en plusieurs parties :
* Variables
* templates
* tâches
* ...

Utiliser `ansible-galaxy` pour créer un échafaudage de rôles de serveur web

```
ansible-galaxy init roles/apache2
```

parcourez le contenu

Nous allons à présent décomposer le précédent fichier que nous avons créé.

* Dans le répertoire `template` du role `apache2`, mettez nos 2 templates précedemment créés
* Nous allons extraire les tâches du playbook précédent et le mettre dans un fichier `installation_apache2.yml` dans le répertoire `tasks`. Puis dans le fichier `main.yml` nous allons inclure une primitive pour incluer la nouvelle tâche ajoutée : 

```yaml
---
# tasks file for roles/apache2
- include_tasks: installation_apache2.yml
```

* Dans le répertoire `handlers`, modifiez le fichier `main.yml` pour y ajouter le handler du playbook précédemment créé

```yaml
---
# handlers file for roles/apache2
- name: redemarrer apache
  service:
    name: apache2
    state: restarted
```

* Dans le répertoire `defaults`, fichier `main.yml` mettre les variables

```yaml
---
  http_port: 8000
  https_port: 4443
  html_msg: "Hello Lab Orsys"
```

### deploiement de notre rôle

Nous allons créer un playbook simple qui contient un play qui déploie notre rôle sur les serveurs du groupe web:

```yaml
- hosts: web
  become: yes

  roles:
    - /lab/roles/apache2
    # On peut aussi tout simplement mettre le nom du rôle étant donné que nous avons renseigné roles_path = /lab/roles 

  tags:
    - web
```

### Utilisation d'un role existant

#### Role mysql

Télécharger le role [geerlingguy.mysql](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) et le placer dans le répertoire `/tp/roles`

```bash
ansible-galaxy role install geerlingguy.mysql -p /tp/roles/
```

* Lister le contenu du répertoire `/tp/roles`; pourquoi avons nous 2 roles ?

Explorer le contenu des deux roles

Créez un playbook qui install ce role sur la cible `db` qui peut être le node3 avec les variables ajustées selon le fait que nous voulons connecter une application WordPress installée sur le node2 à la base de données.

<details><summary>Correction</summary>

```yaml
- hosts: db

  vars:
    mysql_root_password: root
    mysql_databases:
      - name: wordpress
    mysql_users:
      - name: wordpress
        host: '%'
        password: wordpress
        priv: 'wordpress.*:ALL'

  roles:
    - role: geerlingguy.mysql
      become: yes
```

</details>

Tester la conectivité depuis le controller

```bash
sudo apt-get update && sudo apt-get install mysql-client
mysql -u wordpress -p -h node3

mysql> show databases;
```

