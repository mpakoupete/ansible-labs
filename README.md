# ansible labs : Apprendre Ansible par la Pratique

Pour ce cours sur Ansible, nous allons lancer 4 VMs : un noeud de controle Ansible et 3 noeuds cibles:

## Installation de l'environement

**Step 1 :** Installer ces 2 logiciels :
* [Vagrant](https://www.vagrantup.com/downloads)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

**Step 2 :** Télécharger les fichiers du Lab et lancer vos 4 noeuds

* Télécharger le contenu du répertoire `0-setup`
* Placez-vous dans le répertoire où se trouve le contenu
* Ensuite, démarrez l'exécution avec la commande vagrant `vagran up`
* Une fois l'exécution terminée, afficher les VMs installées `vagrant status`

## Description de l'environement de Lab

Nous avons déployé 4 VMs dont une VM de controle Ansible et 3 noeuds
```
controller                running (virtualbox)
node1                     running (virtualbox)
node2                     running (virtualbox)
node3                     running (virtualbox)
```

Nous travaillerons majoritairement sur la machine `controller`. Le répertoire `~/lab_files` sur la machine hôte des VMs est synchronisé avec le répetoire `/lab` sur la VM `controller`. Nous pourront donc dans la suite des Labs éditer les fichiers `YAML` sur la machine hôte et retrouver ces fichiers dans la VM `controller` dans le répertoire `/lab` sans efforts.
