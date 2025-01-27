# Chart Helm de démonstration DSO

## Présentation

Ce chart HELM permet de déployer l'application de [démonstration Java](https://github.com/cloud-pi-native/tuto-java. Il convient de suivre dans un premier temps le tutoriel [démonstration Java](https://github.com/cloud-pi-native/tuto-java) depuis la branche *tuto*

Ce chart permet de créer :
 - Appel au chart HELM par déclaration de dépendance pour la création d'une instance PosgreSQL / Bitnami
 - Création d'un déploiement de l'image construite par DSO sur le projet [démonstration Java](https://github.com/cloud-pi-native/tuto-java)
 - Création d'un service sur le déploiement
 - Création d'un ingress en https vers le service

Dans le cadre de ce tutoriel, nous allons déployer l'application qui a été construite dans le tutoriel précédent.

## Intégration à la chaine CPiN

### Ajout du dépôt externe

Nous allons détailler l'intégration du repo d'infra de démo à l'offre Cloud Pi Native sur la plateforme d'accéleration.

Dans un premier temps il est nécessaire d'ajouter le *repo de code* d'infrastructure au projet contenant la construction de l'applicatoin tuto-java :

1. Depuis son *projet*, aller dans l'onglet *Dépôt*, puis *ajouter un nouveau dépôt* :

 - *Nom du dépôt Git interne* : demo-java-infra

Le repo contient du code d'infrastructure donc cochez la case *Dépôt contenant du code d'infrastructure*.

2. Renseigner *l'URL du repo externe* [https://github.com/cloud-pi-native/tuto-java-infra-helm.git](https://github.com/cloud-pi-native/tuto-java-infra-helm.git). Le repo est public, laissez donc décocher la case *Dépôt de source privé*

Cliquez sur le bouton *Ajouter le dépôt* et attendre que le dépôt apparaisse dans la console.

3. depuis l'onglet *Services externes* vérifier en cliquant sur le service Gitlab que le dépôt *demo-java-infra* est bien présent dans ses projets gitlab.

### Création d'un environnement

Afin de déployer l'application, il est nécessaire de créer un environnement depuis la console. Pour cela, aller dans le menu *Environnements* puis cliquez sur le bouton "+Ajouter un nouvel environnement" :
 - Nom : Donnez un nom logique à l'environnement : demo, dev, integ, prd, etc. Il est conseillé de choisir des noms cours car le nom est utilisé dans les objets Kubernetes créés or ceux-ci sont limité à 63 caractère au total.
 - Choisir une zone : sur l'environnement d'accélération, une seule zone est disponible: *Zone défaut*.
 - Type d'environnement: choisir *dev*. Le choix d'un type d'environnement permet de filtre les dimmensionnements proposés
 - Dimensionnement: choisir *small*. le dimensionnement appose un quota de ressource sur le namespace correspondant au projet.
 - Cluster : choisir le cluster *formation-ovh* (ce cluster est dédié aux exercices et peut être facilement purgé)

Cliquez sur le bouton *Ajouter l'environnement* et attendre que l'environnement soir créé dans la console et apparaisse dans la liste des environnements de son projet.

## Déploiement de l'application

Lorsqu'un projet contient un repo d'infrastructure et (au moins) un environnement, la console crée automatiquement les applications *ArgoCD* associées. Ainsi, depuis le menu gauche  *Services externes* cliquez sur la tuile *ArgoCD DSO* puis le bouton *login via Keycloak* et vérifier que vous retrouvez votre application.

L'application est créée avec un certain nombre de paramètre par défaut qui ne correspondent pas *forcément* à la réalité. Pour cela, depuis l'application, cliquez sur le bouton *Details" en haut à gauche. Ce menu présente les informations principales de l'application ArgoCD :
- Le cluster et le namespace de déploiement
- Le repo Git associé (repo d'infrastructure)
- La branche utilisée sur le repo (par defaut *HEAD*)
- Le répertoire dans lequel chercher les éléments d'infrastructure.

Les éléments à vérifier de façon générales sont : 
 - La branche utilisée, par defaut il s'agit de la branche principale du repo, mais il est possible suivant le cas de modifier cette branche. Par exemple, pour utiliser la branche develop ou dso du projet, il est possible d'éditer cette information pour remplacer *HEAD* par le nom de la branche à utiliser. Pour notre exemple, utilisez la branche tuto.
 - Le répertoire dans lequel chercher les éléments d'infrastructure : Par defaut, la console prépositionne un répertoire *helm* à la racine du projet. Si ce n'est pas le cas il est possible d'éditer cette information pour remplacer *./helm* par le nom du répertoire contenant le code d'infrastructure. A noter que pour utiliser le dossier "racine" du projet il convient de renseigner "*./*". Pour notre exemple, mettez "*./*"

Validez les modifications en cliquant sur le bouton "*SAVE*" en haut à droite.

Attendre quelques secondes et les éléments de déploiement devraient arriver dans l'interface principale d'ArgoCD

## Configuration de l'application

L'application déployée n'est pas fonctionnelle. En effet, un certain nombre d'éléments ne peuvent pas être déduite par avance lors de tutoriel :
 - Emplacement de l'image dans le repository *Harbor* car dépendant du nom du projet.
 - URL de déploiement qui est laissée libre au projet.

Le chart HELM utilisé prévoit déjà le paramétrage de ces éléments, ce qui doit être le cas de tous les charts HELM utilisés.

### Ajouter un fichier values-tuto.yaml

Pour des raisons de facilité, nous allons travailler à partir du repo de code de gitlab et non depuis la source, dans un mode projet, il conviendrait de travailler depuis le repo externe et de procéder à des synchronisation repo externe -> repo interne.

1. Depuis Gitlab, aller dans le projet *demo-java-infra* et choisir la branche *tuto* puis sur le bouton *edit* -> *web IDE* créer un fichier values-tuto.yaml

2. Ajouter le contenu suivant: 
```yaml
image:
  repository: harbor.apps.dso.numerique-interieur.com/form-tuto/java-demo
  pullPolicy: Always
  # Overrides the image tag whose default is the chart appVersion.
  tag: "master"

ingress:
  host: tuto.apps.dso-formation.hp.numerique-interieur.com
```

Adapter le contenu en fonction de votre projet :
 - **image.repository** : correspond à l'emplacement de l'image construite dans le tuto précédent et deployer sur Harbor. Pour connaitre l'emplacement, depuis la console, aller sur le menu *Tableau de bord* de son projet puis cliquez sur le bouton *Afficher les secrets des services* dans le bloc *Harbor* est précisé la racine de déploiement des images du projet, par exemple *harbor.apps.dso.numerique-interieur.com/mi-service-team/*. Attention, il convient de laisser dans *image.repository* le nom de l'image construite (/java-demo)
 - **image.pullPolicy** : Correspond à la politique de téléchargement de l'image lors du déploiement, laisser à la valeur "Always" pour être sûr de récupérer toujours la dernière version de l'image. En mode projet, une politique de tag immutable est fortement conseillée et donc la politique de déploiement pourra passer à *IfNotPresent*. Dans le cadre de ce tuto, laisser *Always*
 - **tag**: tag de l'image associé à **image.repository**, pour le tuto mettre *tuto*
 - **ingress.host** : Nom DNS de l'application. Sur l'environnement d'accélération, la génération des DNS et des certiifcats est automatiquement géré en respectant les sous domaines liés aux clusters. La documentation présente ce point [ici](https://gitlab.apps.dso.numerique-interieur.com/forge-mi/transverse/documentation-dso-projets-interne/-/blob/main/specificite-ovh.md?ref_type=heads). Pour le tutoriel, mettre un nom de la forme <NOM_APPLI>.dso-formation.hp.numerique-interieur.com /!\ Attention /!\  ce nom doit être unique.


Une fois que ce fichier est créé et commit / push sur le repos git de gitlab, retourner sur argoCD sur son application et cliquez sur le bouton "Details". Dans l'écran qui apparait, choisir l'onglet *PARAMETERS* cliqiuez sur le bouton *EDIT* puis dans values files choisir par autocomplétion le fichier values-tuto.yaml.

Sauvegarder et vérifier que le déploiement s'effectue correctement (tout les éléments doivent passer en vert)

### Vérification

Une fois le déploiement terminé et opérationnel, ouvrir un navigateur et vérifier votre l'URL que vous avez saisie dans le fichier *values-tuto.yaml* sur la clé **ingress.host** https://<NOM_APPLI>.dso-formation.hp.numerique-interieur.com/api/demo/demo Si tout est correctement configuré, vous devez avoir une liste au format JSON contenant la liste des personnes présentes en base de données de l'application de tuto.

> Bravo vous avez terminé le tutoriel de déploiement !

Nous allons maintenant voir la gestion des secrets

## Gestion des secrets

Dans l'exemple de déploiement, les secrets sont présents "*en clair*" dans un objet Secret. Ce fichier est dans templates/secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pg-secret
type: Opaque
data:
  PG_PASSWORD: TXlTdXBFclBhU1N3T3Jk
  PG_ROOT_PASSWORD: TXlTdXBFclBhU1N3T3Jk
```

Il est possible de décoder facilement la valeur de ces secrets car ils sont encodé en base64 :

```bash
$ echo TXlTdXBFclBhU1N3T3Jk | base64 -d
MySupErPaSSwOrd
```

Cette pratique ne convient pas à une utilisation en production avec des vrais mots de passe. Ainsi, l'offre Cloud Pi Native propose d"utiliser la solution de chiffrement asymétrique [SOPS](https://github.com/getsops/sops) pour chiffrer les secrets. Le principe est que chaque cluster Cloud Pi Native possèdent sa paire de clé privée / publique. le projet chiffre ses secrets en utilisant la clé publique du cluster et lorsque les fichiers chiffrés sont déployés, le cluster les déchiffre car il connait la clé privée associée.

Pour cela nous allons avoir besoin de l'utilitaire [AGE](https://github.com/FiloSottile/age/releases)

De la documentation complémentaire est présente sur le [repo d'exemple](https://github.com/cloud-pi-native/exemples_ServiceTeam/tree/main/secrets/sops) ou sur la documentation [Cloud Pi Native](https://cloud-pi-native.fr/guide/secrets-management)

### Récupération de la clé publique

La clé publique des différents cluster est présente sur le repo Gitlab transverse de Cloud Pi Native [Secrets](https://gitlab.apps.dso.numerique-interieur.com/forge-mi/transverse/documentation-dso-projets-interne) dans le fichier gestion-secrets.md

Récupérer la clé publique du cluster de formation (age19tfqxgdgx3fe96j8fyy0c65nsfj8ku8sl4ccfxnzpn3xpakylg5s8sgac7)

### Création d'un fichier de secret

Créer un fichier sur son poste, nommé *secret.yaml* dont le contenu est le suivant :

```yaml
apiVersion: isindir.github.com/v1alpha3
kind: SopsSecret
metadata:
  name: tuto-sops-secret
spec:
  secretTemplates:
    - name: pg-secret-sops
      stringData:
        PG_PASSWORD: MySupErPaSSwOrd
        PG_ROOT_PASSWORD: MySupErPaSSwOrd
```
Comme on peut le voir on crée un objet de type SopsSecret et non plus Secret dans le champ *kind*

Lancer la commande suivante :

```bash
sops -e --age age19tfqxgdgx3fe96j8fyy0c65nsfj8ku8sl4ccfxnzpn3xpakylg5s8sgac7 --encrypted-suffix Templates secret.yml > secret.enc.yaml
```

Cette commande va chiffrer le contenu des clés dont le suffix est Templates avec la clé public passée en paramètre et l'écrire dans un fichier *secret.enc.yaml* Il est important de conserver l'extension du fichier donc "*.enc.yaml*" et pas "*.yaml.enc*" car SOPS se base sur cette extension pour déterminer le contenu du fichier.

Le contenu du fichier *secret.enc.yaml* est le suivant :

```yaml
apiVersion: isindir.github.com/v1alpha3
kind: SopsSecret
metadata:
    name: tuto-sops-secret
spec:
    secretTemplates:
        - name: ENC[AES256_GCM,data:o1j2+TbiOhCyHuEfyBQ=,iv:ojK6VrN0DAu/ZhijXo5u2ighYP/+8/qWFfW2LMKHFQ0=,tag:IFcB+WaIb0pcZydt1ZYldA==,type:str]
          stringData:
            PG_PASSWORD: ENC[AES256_GCM,data:3R8RJzIGf549Q0EUPhg/,iv:4O2i99sb13Dynv20G/SVxJBTZ9HoSPbkLQp1XMxP2Vc=,tag:Wbu4+jESe+lZ+L5VN3OhWQ==,type:str]
            PG_ROOT_PASSWORD: ENC[AES256_GCM,data:/JGwlXwWiMF3eFY8InBF,iv:yh9e6uQAikBm+Xs0gleyxSZZ+Oi+v/8ax1J+T51MRnU=,tag:8TFLJyiQH7+0yP/m/hzIUg==,type:str]
sops:
    kms: []
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age:
        - recipient: age19tfqxgdgx3fe96j8fyy0c65nsfj8ku8sl4ccfxnzpn3xpakylg5s8sgac7
          enc: |
            -----BEGIN AGE ENCRYPTED FILE-----
            YWdlLWVuY3J5cHRpb24ub3JnL3YxCi0+IFgyNTUxOSB3UmpTQ2FxQ3F4KzRIM0oy
            Q2p1MlFIR25tdlRhZ05TMTc2ZEJmamxTdGw4CkM0NkFRODFHdGNiMU1vYUdUbWlv
            M1VqSytIQXd0T1ZUdllDaXNlbU5RdncKLS0tIG9HZXoxMEo2eFYzVVBFQ2NuRnZ4
            aTlHR1FPZ2JrRDJDS25sUjFLMmh1VmsKbgYx2nnTXRLj6TIT2B4vAQUAgAnakWGB
            qJYKj6+Vv4N2lfBek9to2VKNv7n9gIa4KaxKWZlk+73hwoi6Z6/ozA==
            -----END AGE ENCRYPTED FILE-----
    lastmodified: "2025-01-27T14:33:22Z"
    mac: ENC[AES256_GCM,data:Yva1sZYkxUD7D9h6DXpb4Ks8SseNp1gUrSKwsaTo2yvoghLfiJCW0fOdRReCVS3SgwqWi2LwjFGihEsHYFclzBqhGjkOX/zNoWkg5QENZ/5DqlJFlEwkeCNJbF+85T+vIJrP8OQ0qN/3i5Wt4qbOZW0mEhZpRRPlgTPwaJqT+Fw=,iv:rGnChbKV0BHpPpTiB7/MbS3nY24gA/sUAY0aLy4TsFI=,tag:cY2OAfn/A3a0jIYYDcMjhA==,type:str]
    pgp: []
    encrypted_suffix: Templates
    version: 3.8.1
```

Comme on peut le voir le contenu du fichier de secret est chiffré : 
```yaml
  PG_PASSWORD: ENC[AES256_GCM,data:3R8RJzIGf549Q0EUPhg/,iv:4O2i99sb13Dynv20G/SVxJBTZ9HoSPbkLQp1XMxP2Vc=,tag:Wbu4+jESe+lZ+L5VN3OhWQ==,type:str]
  PG_ROOT_PASSWORD: ENC[AES256_GCM,data:/JGwlXwWiMF3eFY8InBF,iv:yh9e6uQAikBm+Xs0gleyxSZZ+Oi+v/8ax1J+T51MRnU=,tag:8TFLJyiQH7+0yP/m/hzIUg==,type:str]
```

> Le fichier *secret.yaml* **ne doit pas être commité / push** sur le repo git alors que le fichier *secret.enc.yaml* peut l'être.

Ajouter le fichier  *secret.enc.yaml* dans le répertoire *templates* du repo d'infra et redéployez, puis depuis ArgoCD faite un refresh / sync afin d'actualiser le contenu des éléments déployer. Vous devriez voir l'objet SopsSecret de créé et un objet Secret associé.

Afin de ne pas entrer en collision avec l'objet Secret *pg-secret* déjà existantn, le nom du secret que nous créé est *pg-secret-sops*. Afin d'utiliser ce secret plutot que le fichier *secret.yaml* déjà existant dans le repo, modifier le fichier deployment.yaml pour changer le nom du secret de *pg-secret* à *pg-secret-sops* :
```yaml
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: pg-secret-sops
                  key: PG_PASSWORD
```

Vous pouvez maintenant supprimer le fichier secret.yaml du repo gitlab pour ne laisser que le fichier secret.enc.yaml car il n'est plus référencé. Sous argoCD refaite un refresh / sync pour vérifier que l'objet secret pg-secret a bien été supprimé et que le projet continue de fonctionner.

> Bravo ! Vous savez maintenant comment gérer les secrets via SOPS dans CPiN !

Ceci est la fin du tutoriel de déploiement.