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

Dans le répertoire `/lab` créer un simple fichier `ansible.cfg` qui reprends les valeurs par défaut du fichier de configuration standard `/etc/ansible/ansible.cfg`

Modifiez le fichier de configuration ansible que vous venez de créer avec les paramètres suivants :

* Indiquer le chemin du fichier inventaire `/lab/inventory`
* Indiquer le chemin des roles par défaut `/lab/roles`

Créer le fichier `/lab/inventory` et le répertoire `/lab/roles`.
Créer également le répertoire `/lab/playbooks`qui contiendra tous les playbook que nous allons créer

Testez votre configuration ansible à l'aide de la commande suivante : 

```bash
ansible -m ping all
```

## Configuration des clé SSH - No password

Créez une nouvelle paire de clés SSH qui sera utilisé pour s'authentifier au près des hôtes cibles

```bash
ssh-keygen
```

Création d'un nouvel utilisateur `"admin"` sur les hôtes distants servant d’administration

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