
Exercice 1

Titre: Lister les interfaces réseau d'un hôte

Exercice:

Écrire un playbook Ansible ad-hoc qui liste les interfaces réseau d'un hôte.

Correction:
```bash
ansible -i hosts all -m command -a "ip addr show"
```
Explication:

La commande ip addr show affiche les informations sur les interfaces réseau d'un hôte. La commande ansible -i hosts all -m command -a "ip addr show" exécute cette commande sur tous les hôtes spécifiés dans le fichier hosts.

Exercice 2

Titre: Mettre à jour le mot de passe d'un utilisateur

Exercice:

Écrire un playbook Ansible ad-hoc qui met à jour le mot de passe d'un utilisateur.

Correction:

ansible -i hosts all -m user -a "name=utilisateur new_password=nouveau_mot_de_passe"
Explication:

La commande user modifie les informations d'un utilisateur. La commande ansible -i hosts all -m user -a "name=utilisateur new_password=nouveau_mot_de_passe" modifie le mot de passe de l'utilisateur utilisateur sur tous les hôtes spécifiés dans le fichier hosts.

Exercice 3

Titre: Installer un paquet

Exercice:

Écrire un playbook Ansible ad-hoc qui installe un paquet.

Correction:

ansible -i hosts all -m apt -a "name=paquet"
Explication:

La commande apt installe un paquet. La commande ansible -i hosts all -m apt -a "name=paquet" installe le paquet paquet sur tous les hôtes spécifiés dans le fichier hosts.

Exercice 4

Titre: Démarrer un service

Exercice:

Écrire un playbook Ansible ad-hoc qui démarre un service.

Correction:

ansible -i hosts all -m service -a "name=service state=started"
Explication:

La commande service démarre ou arrête un service. La commande ansible -i hosts all -m service -a "name=service state=started" démarre le service service sur tous les hôtes spécifiés dans le fichier hosts.

Exercice 5

Titre: Arrêter un service

Exercice:

Écrire un playbook Ansible ad-hoc qui arrête un service.

Correction:

ansible -i hosts all -m service -a "name=service state=stopped"
Explication:

La commande service démarre ou arrête un service. La commande ansible -i hosts all -m service -a "name=service state=stopped" arrête le service service sur tous les hôtes spécifiés dans le fichier hosts.

Exercice 6

Titre: Redémarrer un service

Exercice:

Écrire un playbook Ansible ad-hoc qui redémarre un service.

Correction:

ansible -i hosts all -m service -a "name=service state=restarted"
Explication:

La commande service démarre ou arrête un service. La commande ansible -i hosts all -m service -a "name=service state=restarted" redémarre le service service sur tous les hôtes spécifiés dans le fichier hosts.

Exercice 7

Titre: Modifier le contenu d'un fichier

Exercice:

Écrire un playbook Ansible ad-hoc qui modifie le contenu d'un fichier.

Correction:

ansible -i hosts all -m file -a "path=/path/to/file content='nouveau contenu'"
Explication:

La commande file modifie le contenu d'un fichier. La commande ansible -i hosts all -m file -a "path=/path/to/file content='nouveau contenu'" modifie le contenu du fichier /path/to/file sur tous les hôtes spécifiés dans le fichier hosts