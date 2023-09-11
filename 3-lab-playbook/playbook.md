# Playbook Ansible

 Dans ce lab, nous administrerons les serveurs avec de simples playbooks

**Exercice 1:** Créez un playbook Ansible simple qui affiche "Hello Lab!" sur tous les hôtes distants.
Quel module avez vous utilisé ? consultez sa documentation.


<details><summary> Correction 1:</summary>

```yaml
---
- name: Exercice 1 - Hello Lab
  hosts: all
  tasks:
    - name: Afficher un message
      debug:
        msg: "Hello Lab!"
```

</details>


**Exercice 2:** Organisez l'inventaire pour mettre node1 & node2 dans un groupe `"web_servers"` et node3 dans le groupe `"db_servers"`. Puis, créez un playbook qui installe le package "git" sur le groupe d'hôtes nommé "web_servers".
Quel module avez-vous utilisé ?
Consultez la documentation et donnez l'avantage ou inconvenient de ce dernier.


<details><summary> Correction 2:</summary>

```yaml
# On peut aussi utiliser le module package ou apt.

# Le module package cache les détails spécifiques à la distribution. est plus générique et fonctionne sur plusieurs distributions Linux. L'inconvenient c'est : Moins de contrôle spécifique à la distribution, Manque de personnalisation avancée

# Le module apt : spécifiques à la distribution Debian,On peut accéder à des fonctionnalités spécifiques, fonctionnalités avancées à la distribution. Son inconvenient c'est : Moins de portabilité, plus de travail de maintenance si vous gérez des serveurs sur plusieurs distributions, 
---
- name: Exercice 2 - Installation de Git
  hosts: web_servers
  tasks:
    - name: Installer Git
      apt:
        name: git
        state: present
```

</details>


**Exercice 3.1:** Créez un playbook qui crée un utilisateur nommé "ansible_user" sur tous les hôtes distants.


<details><summary> Correction 3:</summary>

```yaml
---
- name: Exercice 3.1 - Création d'utilisateur
  hosts: all
  tasks:
    - name: Créer un utilisateur
      user:
        name: ansible_user
```

</details>


**Exercice 3.2:** Créez un playbook qui crée un utilisateur nommé "labuser" sur tous les hôtes distants dont les carracteristiques sont les suivantes :
* uid : 1999
* comment: "Utilisateur Lab"
* appartenant au groupe secondaire : developpeurs
* mot de passe : labuser001

Pour l'instant créez avec la commande Ad-hoc le groupe `developpeurs`. Plus tard, nous verrons comment inclure des contrôles

<details><summary> Correction 3:</summary>

```yaml
---
- name: Création de l'utilisateur labuser
  hosts: all
  tasks:
    - name: Créer l'utilisateur labuser
      user:
        name: labuser
        uid: 1999
        comment: "Utilisateur Lab"
        groups: developpeurs
        password: "{{ 'labuser001' | password_hash('sha512', 'mysecretsalt') }}"
```

</details>


**Exercice 4:** Créez un playbook qui crée un répertoire "/var/www/html" sur un groupe d'hôtes nommé "web_servers".


<details><summary> Correction 4:</summary>

```yaml
---
- name: Exercice 4 - Création de répertoire
  hosts: web_servers
  tasks:
    - name: Créer un répertoire
      file:
        path: /var/www/html
        state: directory
```

</details>


**Exercice 5:** Créez un playbook qui copie un fichier local "index.html" vers le répertoire "/var/www/html" de tous les hôtes distants.
Créez ce fichier index qui contient un simple text html de votre choix

<details><summary> Correction 5:</summary>

```yaml
---
- name: Exercice 5 - Copie de fichier
  hosts: all
  tasks:
    - name: Copier un fichier
      copy:
        src: /chemin/vers/index.html
        dest: /var/www/html/
```

</details>


**Exercice 6:** Créez un playbook qui modifie le fichier "/etc/ssh/sshd_config" pour autoriser l'authentification par clé publique seulement.


<details><summary> Correction 6:</summary>

```yaml
---
- name: Exercice 6 - Configuration SSH
  hosts: all
  tasks:
    - name: Modifier le fichier SSHD
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
```

</details>


**Exercice 7:** Créez un playbook qui redémarre le service Apache sur un groupe d'hôtes nommé "web_servers".


<details><summary> Correction 7:</summary>

```yaml
---
- name: Exercice 7 - Redémarrage Apache
  hosts: web_servers
  tasks:
    - name: Redémarrer Apache
      service:
        name: apache2
        state: restarted
```

</details>


**Exercice 8:** Créez un playbook qui installe un serveur MySQL sur un groupe d'hôtes nommé "db_servers" avec une configuration spécifique.


<details><summary> Correction 8:</summary>

```yaml
---
- name: Exercice 8 - Installation MySQL
  hosts: db_servers
  tasks:
    - name: Installer MySQL Server
      apt:
        name: mysql-server
        state: present
    - name: Copier le fichier de configuration MySQL
      copy:
        src: /chemin/vers/my.cnf
        dest: /etc/mysql/my.cnf
```

</details>


**Exercice 9:** Créez un playbook qui crée un fichier de configuration à partir d'un modèle (template) et le déploie sur un groupe d'hôtes nommé "app_servers".


<details><summary> Correction 9:</summary>

```yaml
---
- name: Exercice 9 - Déploiement de configuration
  hosts: app_servers
  tasks:
    - name: Copier le fichier de configuration à partir d'un modèle
      template:
        src: /chemin/vers/template.conf.j2
        dest: /etc/app/app.conf
```

</details>


**Exercice 10:** Créez un playbook qui effectue une mise à jour complète du système sur tous les hôtes distants.


<details><summary> Correction 10:</summary>

```yaml
---
- name: Exercice 10 - Mise à jour du système
  hosts: all
  tasks:
    - name: Mise à jour du système
      apt:
        upgrade: dist
```

</details>


**Exercice 11:** Créez un playbook qui installe un serveur web Nginx sur un groupe d'hôtes nommé "web_servers" et configure un site web simple.


<details><summary> Correction 11:</summary>

```yaml
---
- name: Exercice 11 - Installation de Nginx
  hosts: web_servers
  tasks:
    - name: Installer Nginx
      apt:
        name: nginx
        state: present
    - name: Copier la configuration du site web
      template:
        src: /chemin/vers/nginx-site.conf.j2
        dest: /etc/nginx/sites-available/my-site
    - name: Activer le site web
      command: ln -s /etc/nginx/sites-available/my-site /etc/nginx/sites-enabled/
    - name: Redémarrer Nginx
      service:
        name: nginx
        state: restarted
```

</details>


**Exercice 12:** Créez un playbook qui crée un utilisateur "lab1" avec des privilèges sudo sur tous les hôtes distants.


<details><summary> Correction 12:</summary>

```yaml
---
- name: Exercice 12 - Création d'utilisateur avec sudo
  hosts: all
  tasks:
    - name: Créer un utilisateur
      user:
        name: lab1
        state: present
    - name: Ajouter l'utilisateur au groupe sudo
      user:
        name: lab1
        groups: sudo
```

</details>


**Exercice 13:** Créez un playbook qui effectue une sauvegarde complète du répertoire "/var/www" sur un serveur distant et la stocke dans "/backup".


<details><summary> Correction 13:</summary>

```yaml
---
- name: Exercice 13 - Sauvegarde du répertoire
  hosts: serveur_de_sauvegarde
  tasks:
    - name: Créer le répertoire de sauvegarde
      file:
        path: /backup
        state: directory
    - name: Effectuer la sauvegarde à l'aide de tar
      command: tar -cz ...
```

```yaml
---
- name: Exercice 13 - Sauvegarde du répertoire
  hosts: serveur_de_sauvegarde
  tasks:
    - name: Créer le répertoire de sauvegarde
      file:
        path: /backup
        state: directory
    - name: Sauvegarder le répertoire /var/www
      archive:
        src: /var/www
        dest: /backup/web-backup-{{ ansible_date_time.iso8601 }}.tar.gz
        compression: gz

```

</details>


**Exercice 14:** Créez un playbook qui crée un utilisateur nommé "labuser2" sur tous les hôtes distants dont les carracteristiques sont les suivantes :
* uid : 2000
* comment: "Utilisateur Lab 2"
* appartenant au groupe secondaire : devops
* mot de passe : labuser002

Nous supposons que le group devops n'est pas encore créé. Inclure dans le playbook un controle qui :
* tente de créer le groupe `devops` dont le gid est `1500`
* si le groupe existe déjà que la sortie soit considéré comme une erreur et que la tâche soit omise (skip)
* ajouter l'utilisateur précédent avec ce nouveau groupe créé

Pour l'instant créez avec la commande Ad-hoc le groupe `developpeurs`. Plus tard, nous verrons comment inclure des contrôles

<details><summary> Correction 14:</summary>

```yaml
---
- name: Création de l'utilisateur labuser
  hosts: all
  tasks:
    - name: Vérifier l'existence du groupe devops
      group:
        name: devops
        gid: 1500
      register: group_result
      ignore_errors: yes

    - name: Créer le groupe devops s'il n'existe pas
      group:
        name: devops
      when: group_result.failed

    - name: Créer l'utilisateur labuser
      user:
        name: labuser
        uid: 1999
        comment: "Utilisateur Lab"
        groups: devops
        password: "{{ 'labuser001' | password_hash('sha512', 'mysecretsalt') }}"

```

</details>

