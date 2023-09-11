
# Ansible Ad-Hoc

Structure d'une commande Ad-Hoc :

```bash
$ ansible [hôte-cible] [inventaire] -m [module] -a "[Arguments du module]" (-b : optionnel pour les commande à privilège)
```

Exemple :

```bash
# ping du node 1
$ ansible node1 -i /lab/inventory -m ping

# si le fichier inventaire est renseigné dans le fichier ansible.cfg on peut se passer de le préciser
$ ansible node1 -m ping
$ ansible -m ping node1

# Commande à privilège
$ ansible all -i /lab/inventory -m file -a "path=/root/ansible_test state=directory" -b
```

**Exercice 1:** Exécutez une commande sur tous les hôtes distants pour obtenir leur date et heure actuelles.

<details><summary> Correction 1:</summary>

```bash
ansible all -i /lab/inventory -m command -a "date"
```

</details>


**Exercice 2:** Obtenez des informations sur l'utilisation de la mémoire RAM de tous les hôtes distants.

<details><summary> Correction 2:</summary>

```bash
ansible all -i /lab/inventory -m command -a "free -m"
```

</details>


**Exercice 3:** Créez un répertoire nommé "ansible_test" dans le répertoire /tmp de tous les hôtes distants.

<details><summary> Correction 3:</summary>

```bash
ansible all -i /lab/inventory -m file -a "path=/tmp/ansible_test state=directory"
```

</details>


**Exercice 4:** Installez le package "nginx" sur tous les hôtes distants.

<details><summary> Correction 4:</summary>

```bash
ansible all -i /lab/inventory -m apt -a "name=nginx state=present" -b
```

</details>


**Exercice 5:** Redémarrez le service Nginx sur tous les hôtes distants.

<details><summary> Correction 5:</summary>

```bash
ansible all -i /lab/inventory -m service -a "name=nginx state=restarted" -b
```

</details>


**Exercice 6:** Créez un fichier texte nommé "hello.txt" avec le contenu "Hello, Ansible!" dans le répertoire /tmp de tous les hôtes distants.

<details><summary> Correction 6:</summary>

```bash
ansible all -i /lab/inventory -m copy -a "content='Hello, Ansible!' dest=/tmp/hello.txt" -b
```

</details>


**Exercice 7:** Ajoutez un utilisateur nommé "ansible_user" sur tous les hôtes distants.

<details><summary> Correction 7:</summary>

```bash
ansible all -i /lab/inventory -m user -a "name=ansible_user" -b
```

</details>


**Exercice 8:** Supprimez le répertoire "ansible_test" du répertoire /tmp de tous les hôtes distants.

<details><summary> Correction 8:</summary>

```bash
ansible all -i /lab/inventory -m file -a "path=/tmp/ansible_test state=absent" -b
```

</details>


**Exercice 9:** Modifiez le fichier "/etc/ssh/sshd_config" sur tous les hôtes distants pour autoriser l'authentification par mot de passe (PasswordAuthentication yes).

<details><summary> Correction 9:</summary>

```bash
ansible all -i /lab/inventory -m lineinfile -a "path=/etc/ssh/sshd_config line='PasswordAuthentication yes'" -b
```

</details>


**Exercice 10:** Mettez à jour tous les packages sur les hôtes distants.

<details><summary> Correction 10:</summary>

```bash
ansible all -i /lab/inventory -m apt -a "upgrade=dist" -b
```

</details>


**Exercice 11:** Créez un utilisateur nommé "webadmin" avec un répertoire personnel sur tous les hôtes distants.

<details><summary> Correction 11:</summary>

```bash
ansible all -i /lab/inventory -m user -a "name=webadmin createhome=yes" -b
```

</details>


**Exercice 12:** Copiez le fichier "index.html" depuis votre machine locale vers le répertoire /var/www/html sur tous les hôtes distants.

<details><summary> Correction 12:</summary>

```bash
ansible all -i /lab/inventory -m copy -a "src=/path/to/index.html dest=/var/www/html/index.html" -b
```

</details>


**Exercice 13:** Ajoutez tous les hôtes distants `node1` & `node2` à un groupe nommé "web_servers" dans le fichier d'inventaire.
et sur les hôtes du groupe web_servers créer un group `web_servers` pour les administrateurs.

<details><summary> Correction 13:</summary>


```bash
vim /lab/inventaire
node3
[web_servers]
node1
node2
```

```bash
ansible web_servers -m group -a "name=web_servers state=present"
ansible web_servers -m user -a "name=admin groups=web_servers"
```

</details>


**Exercice 14:** Vérifiez l'espace disque disponible sur tous les hôtes distants.

<details><summary> Correction 14:</summary>

```bash
ansible all -i /lab/inventory -m command -a "df -h"
```

</details>


**Exercice 15:** Créez un lien symbolique "/var/www/html/latest" vers le fichier "/var/www/html/index.html" sur tous les hôtes distants.

<details><summary> Correction 15:</summary>

```bash
ansible all -i /lab/inventory -m file -a "src=/var/www/html/index.html dest=/var/www/html/latest state=link" -b
```

</details>


**Exercice 16:** Ajoutez un utilisateur nommé "admin" au groupe "sudo" sur tous les hôtes distants.

<details><summary> Correction 16:</summary>

```bash
ansible all -i /lab/inventory -m user -a "name=admin append=yes groups=sudo" -b
```

</details>


**Exercice 17:** Redémarrez tous les hôtes distants.

<details><summary> Correction 17:</summary>

```bash
ansible all -i /lab/inventory -m command -a "reboot" -b
```

</details>


**Exercice 18:** Affichez le contenu du fichier "/etc/hostname" sur tous les hôtes distants.

<details><summary> Correction 18:</summary>

```bash
ansible all -i /lab/inventory -m command -a "cat /etc/hostname"
```

</details>


**Exercice 19:** Créez un répertoire nommé "logs" dans le répertoire /var/www/html sur tous les hôtes distants.

<details><summary> Correction 19:</summary>

```bash
ansible all -i /lab/inventory -m file -a "path=/var/www/html/logs state=directory" -b
```

</details>


**Exercice 20:** Changez le mot de passe de l'utilisateur "ansible_user" sur tous les hôtes distants.

<details><summary> Correction 20:</summary>

```bash
ansible all -i /lab/inventory -m user -a "name=ansible_user update_password=always" -b
```

</details>
