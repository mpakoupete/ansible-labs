# Création de module Ansible

Dans ce laboratoire, vous allez créer un module Ansible personnalisé qui interroge une API pour récupérer des informations sur un aéroport spécifique en utilisant son code [ICAO](https://www.world-airport-codes.com/) (Ex: `LFPG` pour l'aeroport Charles de Gaulle). Vous apprendrez à gérer les entrées utilisateur, à effectuer des appels API et à traiter les réponses JSON.

Vous pouvez l'adapter à n'importe quel autre API. [Ici](https://mixedanalytics.com/blog/list-actually-free-open-no-auth-needed-apis/) vous trouverez une liste d'APIs gratuite que vous pouvez utiliser dans ce lab.

Dans la suite du lab nous utiliserons **[Aviation Weather Center](https://aviationweather.gov/data/api/)** en particulier `/api/data/airport`

* Exemple : https://aviationweather.gov/api/data/airport?ids=LFPG&format=json

## Objectif:

L'objectif du module que nous allons créer est de prendre comme argument obligatoire `airport_code` et qui retourne les informations au format JSON.
Le nom du module s'appelera : `get_airport_info`


## Création du Module Ansible:

Pour que le module soit appelable natiement par Ansible, il faut qu'il soit dans un répertoire de modules.
* Pour afficher les répertoires de modules vous pouvez exécuter la commande suivante et placer le module dans l'un des répertoires
```bash
ansible-config dump | grep -i MODULE_PATH
```
* Ou bien vous pouvez ajouter un répertoire contenant les modules dans le fichier `ansible.cfg`
```bash
# (pathspec) Colon-separated paths in which Ansible will search for Modules.
library=/home/vagrant/.ansible/plugins/modules:/usr/share/ansible/plugins/modules:/path/to/my/new/modules/dir
```

* Créez un fichier Python nommé `get_airport_info.py`.

* Utilisez la classe `AnsibleModule` pour définir un argument `airport_code` qui est requis.

* Écrivez une fonction qui fait une requête GET à l'API `https://aviationweather.gov/api/data/airport?ids=$(airport_code)&format=json` pour récupérer les informations de l'aéroport.

* Retournez les informations de l'aéroport récupérées ou un message d'erreur si la requête échoue.

## Utilisation du Module dans un Playbook:**

* Créez un playbook Ansible dans le répertoire `playbook`.

* Ajoutez une tâche qui utilise votre module personnalisé pour récupérer les informations de l'aéroport en utilisant par exemple le code `LFPG`.

* Enregistrez la réponse dans une variable nommée `airport_info`.

* Ajoutez une tâche supplémentaire pour extraire et afficher uniquement le nom de l'aéroport à partir de la variable `airport_info`.

## Gestion des Erreurs:

* Testez votre module avec un code d'aéroport invalide pour vous assurer que les erreurs sont gérées correctement.
* Modifiez le playbook pour afficher un message d'erreur personnalisé si le module échoue.

## Resources

* Documentation Ansible sur la création de modules: https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html
* Documentation Python sur la gestion des requêtes HTTP: https://docs.python.org/3/library/urllib.request.html#module-urllib.request

## Solution

<details><summary>code du module Python</summary>

```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule
import json
try:
    from urllib.request import urlopen
except ImportError:
    from urllib2 import urlopen

def get_airport_info(airport_code):
    url = "https://aviationweather.gov/api/data/airport?ids={}&format=json".format(airport_code)
    response = urlopen(url)
    data = response.read()
    return json.loads(data)

def main():
    module_args = dict(
        airport_code=dict(type='str', required=True),
    )
    result = dict(
        changed=False,
        info={},
    )
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )
    if module.check_mode:
        module.exit_json(**result)
    try:
        result['info'] = get_airport_info(module.params['airport_code'])
        module.exit_json(**result)
    except Exception as e:
        module.fail_json(msg=str(e))
if __name__ == '__main__':
    main()
```

</details>

<details><summary>Exemple d'utilisation du module sera donc</summary>

```yaml
- hosts: node1
  gather_facts: no

  tasks:
    - name: Obtenir les informations de l'aeroport
      get_airport_info:
        airport_code: "LFPG"
      register: airport_info

    - name: Enregistrer un fact pour la première donnée
      set_fact:
        first_airport_name: "{{ airport_info.info[0].name }}"

    - name: Afficher le nom de l'aeroport
      debug:
        var: first_airport_name
```

```yaml
- hosts: node1
  gather_facts: no

  tasks:
    - name: Obtenir les informations de l'aeroport
      get_airport_info:
        airport_code: "LFPG"
      register: airport_info

    - name: Afficher tout le contenu de la variable airport_info
      debug:
        var: airport_info
```

</details>