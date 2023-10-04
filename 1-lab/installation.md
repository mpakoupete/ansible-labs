# Configuration de notre infrastructure

Afin de pouvoir gérer notre parc de serveur avec ansible, il nous faudra :

* Installer Ansible
* Configuer les clé SSH pour éviter de resaisir les mot de passes à chaque exécution
* Tester

## Installation de Ansible

Suivre les étapes suivantes ([guide officiel](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip)) pour l'installation de Ansible sur la VM **controller**

Sur la machine de Lab, connectez vous à la VM controller :

```bash
vagrant ssh controller
```

Pour vérifier si pip est déjà installé pour votre Python préféré :
```bash
python3 -m pip -V
```

Si `pip` n'est pas installé, veuillez l'installer à l'aide des commandes suivantes:

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
```

Utilisez pip dans l'environnement Python que vous avez choisi pour installer le paquetage complet d'Ansible pour l'utilisateur actuel :

```bash
python3 -m pip install --user ansible
```

## Configuration de base

Dans le répertoire personnel de l'utilisateur `vagrant`, créer un répertoire `lab`. Tout le reste des fichiers sera créé à l'intérieur.

- Généré/créer le fichier de configuration `ansible.cfg` qui reprends les valeurs par défaut de la config ansible. _(sur une installation avec apt/yum le fichiers de configuration standard se trouve `/etc/ansible/ansible.cfg`)_

Modifiez le fichier de configuration ansible que vous venez de créer avec les paramètres suivants :

* Indiquer le chemin du fichier inventaire `/home/vagrant/lab/inventory`
* Indiquer le chemin des roles par défaut `/home/vagrant/lab/roles`

Créer le fichier `/home/vagrant/lab/inventory` et le répertoire `/home/vagrant/lab/roles`.
Créer également le répertoire `/home/vagrant/lab/playbooks`qui contiendra tous les playbook que nous allons créer

Testez votre configuration ansible à l'aide de la commande suivante : 

```bash
ansible -m ping all
```

## Configuration des clé SSH - No password

Sur la machine de control `controller`, créez une nouvelle paire de clés SSH qui sera utilisé par l'utilisateur `vagrant` pour s'authentifier au près des hôtes cibles : `node1`, `node2`, `node3`

```bash
# Generation de clé ssh
ssh-keygen
```

Sur les hôtes cibles : `node1`, `node2`, `node3`, créez un nouvel utilisateur `"admin"` qui sera utiliser pour d’administration

* Sur chaque hôte; créer manuellement un utilisateur `"admin"`

```bash
useradd -m "admin"

# donnez un mot de passe (e.g. adminpass)
passwd admin
```

* Ajouter à l'utilisateur `admin` les capacités **sudo**.

```bash
cat << _EOF > /etc/sudoers.d/admin
admin         ALL=(ALL)       NOPASSWD:ALL
_EOF
# NOPASSWD est une indication pour ne pas demander de mot de passe à chaque élevation de privilège
```

Depuis le serveur `controller`, exportez votre clé SSH

```bash
ssh-copy-id admin@node1
ssh-copy-id admin@node2
ssh-copy-id admin@node3
```

Tester une commande Ad-Hoc Ansible simple (e.g. le Ping)

```bash
ansible all -u admin -m ping
```

Tester une commande à privilège avec l'escalade de privilège; on ne devrait plus vous demander aucun mot de passe : 

```bash
ansible all -u admin -b -m user -a "name=testuser state=present"
```

```bash
ansible all -u admin -b -m user -a "name=testuser state=absent"
```

Modifiez à présent le fichier de configuration `ansible.cfg` pour que l'utilisateur par défaut soit `admin`. Faites le test

```bash
ansible all -m ping
ansible all -b -m ping
```

<details><summary> Autre solution </summary>

Dans un parc de dizaine, voir de centaines de serveurs il est plus judicieux de simplifier les étapes précendentes. On peut le faire de plusiseurs manières:

- Lors de la création des hôtes cibles, créer un utilisateur `admin` et mettre dans le fichier `~/.ssh/authorized_keys` la clé ou les clés publics de/des administrateurs. _(la methode la plus judicieuse)_
- Lors de la création des hôtes cibles, utiliser l'utilisateur par défaut `root` connaissant son mot de passe pour faire la configuration de l'utilisateur d'administration Ansible. Les commandes suivante montrent comment on peut le faire directement à partir de la machine `controller` sans passer serveur par serveur.

```bash
ansible all -u root -m user -a "name=admin state=present"
```

Exportation de notre clé Publique vers les hôtes administrés

```bash
ansible all -u root -m authorized_key -a "user=admin state=present key='{{ lookup('file', '/home/vagrant/.ssh/id_rsa.pub') }}'"
```

Modification du fichier `"/etc/sudoers"` afin que l’escalade de privilège ne requiert pas de mot de passe

```bash
ansible all -u root -m lineinfile -a "path=/etc/sudoers insertafter='^root' line='admin    ALL=(ALL)      NOPASSWD:ALL'"
```

Tester une commande à privilège avec l'escalade de privilège; on ne devrait plus vous demander aucun mot de passe : 

```bash
ansible all -u admin -b -m user -a "name=testuser state=present"
```

```bash
ansible all -u admin -b -m user -a "name=testuser state=absent"
```

</details>

