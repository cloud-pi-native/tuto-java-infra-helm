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

### Déploiement de l'application

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

### Configuration de l'application

L'application déployée n'est pas fonctionnelle. En effet, un certain nombre d'éléments ne peuvent pas être déduite par avance lors de tutoriel :
 - Emplacement de l'image dans le repository *Harbor* car dépendant du nom du projet.
 - URL de déploiement qui est laissée libre au projet.

Le chart HELM utilisé prévoit déjà le paramétrage de ces éléments, ce qui doit être le cas de tous les charts HELM utilisés.

#### Ajouter un fichier values-tuto.yaml

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

#### Vérification

Une fois le déploiement terminé et opérationnel, ouvrir un navigateur et vérifier votre l'URL que vous avez saisie dans le fichier *values-tuto.yaml* sur la clé **ingress.host** https://<NOM_APPLI>.dso-formation.hp.numerique-interieur.com/api/demo/demo Si tout est correctement configuré, vous devez avoir une liste au format JSON contenant la liste des personnes présentes en base de données de l'application de tuto.

> Bravo vous avez terminé le tutoriel de déploiement !