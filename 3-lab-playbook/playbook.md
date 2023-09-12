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


**Exercice 5:** Créez un playbook qui copie un fichier local "index.html" vers le répertoire "/var/www/html" des hôtes du groupe "web_servers"..
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


**Exercice 7:** Créez un playbook qui : 
* install Apache
* redémarre le service Apache sur le groupe d'hôtes nommé "web_servers"
* qui enregistre la sortie de la  tâche 2 `Redémarrer Apache` dans une variable nommée `apache_restat_output` (utilisez le plugin `register`)
* Une troisième tâche qui affiche le code retour de cette variable (utilisez `debug`)

Le playbook s'est'il exécuté correctement ?
La Troisième tâche a t'elle pu s'exécuter ?
Si non pourquoi ?


<details><summary> Correction 7:</summary>

```yaml
---
- name: Exercice 7 - Redémarrage Apache
  hosts: web_servers
  tasks:
    - name: Installation Apache
      apt:
        name: apache2
        state: latest
    - name: Redémarrer Apache
      service:
        name: apache2
        state: restarted
      register: apache_restat_output
    - name: affiche retour tache 2
      debug:
        msg: apache_restat_output
```

non car nginx utilise déjà le Port 80 et donc Apache n'a pas pu se lancer. Il faut diagnostiquer dans les logs pour voir ce conflit.
Et quand une tâche échoue le reste des tâches s'intérompt. Raison pour laquelle il n'a pas exécuté la tâche 3

</details>

**Exercice 7.2:** Modifiez le playbook précedant pour :
* Ignorer l'erreur en cas d'échec.
* Et ajouter une 4ème tâche qui indique qu'il y'a un message d'erreur si la tâche 2 se s'exécute pas bien. (utiliser `debug` et la condition `when`)


<details><summary> Correction 7:</summary>

```yaml
---
- name: Exercice 7 - Redémarrage Apache
  hosts: web_servers
  tasks:
    - name: Installation Apache
      apt:
        name: apache2
        state: latest
    - name: Redémarrer Apache
      service:
        name: apache2
        state: restarted
      register: apache_restat_output
      ignore_errors: true
    - name: affiche le retour de la tache 2
      debug:
        msg: '{{ apache_restat_output }}'
    - name: Dire si la tâche a échouée
      debug:
        msg: "La tâche de redémarrage a échoué"
      when:
        - apache_restat_output.failed == true
```

non car nginx utilise déjà le Port 80. Il faut diagnostiquer dans les logs pour voir ce conflit.

</details>

**Exercice 8:** Créez un playbook qui installe un serveur MySQL sur un groupe d'hôtes nommé "db_servers" avec une configuration spécifique contenu dans un fichier `my.cnf` et le redémarer.


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
        update_cache: true
    - name: Copier le fichier de configuration MySQL
      copy:
        src: /chemin/vers/my.cnf
        dest: /etc/mysql/my.cnf
    - name: redémarrer mysql
      service:
        name: mysql
        state: restarted
```

</details>


**Exercice 9:** Créez un playbook qui crée un fichier de configuration nommé `/etc/app/app.conf` à partir d'un modèle (template) et le déploier sur le groupe d'hôtes nommé "web_servers". Faites en sorte que les valeurs mises entre `< >` soit automatiquement mise selon la machine sur laquelle la conf sera mise. (Ex: pour le node1, que l'IP soit l'IP du node1)

```
# Sample conf
Adresse IP : <IP du Serveur>
Nom du serveur : <Hostname>
```


<details><summary> Correction 9:</summary>

Créer le template suivant

```
# Sample conf
Adresse IP : {{ ansible_default_ipv4.address }}
Nom du serveur : {{ ansible_hostname }}
```

```yaml
---
- name: Exercice 9 - Déploiement de configuration
  hosts: web_servers
  tasks:
    - name: créer le répertoire /etc/app
      file:
        path: /etc/app
        state: directory
    - name: Copier le fichier de configuration à partir d'un modèle
      template:
        src: /lab/template.j2
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


**Exercice 11:** Créez un playbook qui crée un utilisateur "lab1" avec des privilèges sudo sur tous les hôtes distants.


<details><summary> Correction 11:</summary>

```yaml
---
- name: Exercice 11 - Création d'utilisateur avec sudo
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


**Exercice 12:** Créez un playbook qui effectue une sauvegarde complète du répertoire "/var/www" et la stocke dans le répertoire "/backup". Faite en sorte que le nom des archives créées contienne la date et l'heure exacte de la création de l'archive format : "var_www_archive_`YYYY-MM-DD_HH-mm-ss`.tar.gz"


<details><summary> Correction 12:</summary>

```yaml
---
- name: Exercice 12 - Sauvegarde du répertoire
  hosts: web_servers
  tasks:
    - name: Obtenir le temps présent
      set_fact:
        timestamp: "{{ ansible_date_time.date }}-{{ ansible_date_time.time | regex_replace(':', '-') }}"
    - name: Créer le répertoire de sauvegarde
      file:
        path: /backup
        state: directory
    - name:
      archive:
        path: /var/www
        dest: "/backup/var_www_archive_{{ timestamp }}.tar.gz"
        format: gz
        mode: '0644'
```

</details>


**Exercice 13:** Créez un playbook qui crée un utilisateur nommé "labuser2" sur tous les hôtes distants dont les carracteristiques sont les suivantes :
* uid : 2000
* comment: "Utilisateur Lab 2"
* appartenant au groupe secondaire : devops
* mot de passe : labuser002

Nous supposons que le group devops n'est pas encore créé. Inclure dans le playbook un controle qui :
* tente de créer le groupe `devops` dont le gid est `1500`
* si le groupe existe déjà que la sortie soit considéré comme une erreur et que la tâche soit omise (skip)
* ajouter l'utilisateur précédent avec ce nouveau groupe créé

Pour l'instant créez avec la commande Ad-hoc le groupe `developpeurs`. Plus tard, nous verrons comment inclure des contrôles

<details><summary> Correction 13:</summary>

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



**Exercice 14:** 

Créez la liste des utilisateurs suivants au format yaml dans un dictionnaire du nom de `users` et enregistrez le dans un fichier `liste_utilisateurs.yml`:

* username: lab1; uid: 2001
* username: lab2; uid: 2002
* username: lab3; uid: 3003
* username: lab4; uid: 3004

<details><summary>Correction Liste utilisateurs</summary>

```yaml
---
 users:
  - username: lab1
    uid: 2001
  - username: lab2
    uid: 2002
  - username: lab3
    uid: 3003
  - username: lab4
    uid: 3004
```

</details>

Créer un fichier nommé `secret.yml` contenant une variable nommé `user_passwd : p@SSw0duser`
Chiffrez le avec ansible-vault 

<details><summary>Correction Secret.yml:</summary>

```yaml
---
  user_passwd: p@SSw0duser
  
```

```
# chriffrer ==> entrez le mot de passe
ansible-vault encrypt /lab/secret.yml

# voir le contenu
cat /lab/secret.yml

# voir le contenu décrypté
ansible-vault view /lab/secret.yml
```

</details>


Créer un playbook qui importe les fichiers `liste_utilisateurs.yml` et `secret.yml` comme variable (utiliser `var_files`) et créer ces utilisateurs avec les caractériqtiques suivantes :
* les noms et uid correspondent aux variables contenues dans `liste_utilisateurs.yml`
* Le mot de passe de tous les utilisateurs est le même et qui est contenu dans  `secret.yml`
* Les utilisateurs dont les UID sont inférieures à 3000 seront créés sur les hôtes appartenant au groupe `web_servers`
* Et les utilisateurs dont les UID sont supérieurs à 3000 seront créés sur les hôtes appartenant au groupe `db_servers` 

<details><summary> Correction Playbook: </summary>

```yaml
---
 - hosts : all
   become : yes

   vars_files :
    - secret.yml
    - liste_utilisateurs.yml

   tasks :
    - name : Création des utilisateurs sur Web_servers
      user : 
        name : "{{ item.username }}"
        uid : "{{ item.uid }}"
        password : "{{ user_passwd | password_hash('sha512') }}"
      with_items :
        - "{{ users }}"
      when :
        - inventory_hostname in groups['web_servers'] and item.uid|int < 3000
      
    - name : Création des utilisateurs sur db_servers
      user : 
        name : "{{ item.username }}"
        uid : "{{ item.uid }}"
        password : "{{ user_passwd | password_hash('sha512') }}"
      with_items :
        - "{{ users }}"
      when :
        - inventory_hostname in groups['db_servers'] and item.uid|int > 2999
  
```

Afin de pouvoir exécuter notre ansible-playbook, nous devons lui fournir le mot de passe vault pour déchiffrer le le fichier secret.yml
on met notre mot de passe de chiffrement dans un fichier
```
vim vault_password
```


Ensuite on ajoute cela comme argument à la commande

```
ansible-playbook /lab/exo14.yml --vault-password-file vault_password
```

</details>