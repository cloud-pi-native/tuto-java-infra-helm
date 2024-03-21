# Chart Helm de démonstration DSO

## Présentation

Ce chart HELM permet de déployer l'application de [démonstration Java](https://github.com/dnum-mi/dso-tuto-java)

Ce chart permet de créer :
 - Appel au chart HELM par déclaration de dépendance pour la création d'une instance PosgreSQL / Bitnami
 - Création d'un déploiement de l'image construite par DSO sur le projet [démonstration Java](https://github.com/dnum-mi/dso-tuto-java)
 - Création d'un service sur le déploiement
 - Création d'un ingress en https vers le service

Ce Chart peut servir dans le cas d'un test de déploiement comme repo d'infrastructure depuis la console DSO.

## Mode opératoire

Le plus simple est de réaliser un fork de ce repo github vers son organisation afin de pouvoir le modifier facilement.

Une fois le repo forké dans son espace, éditer le fichier de values.yaml et modifier les 2 variables suivantes :

```yaml
image:
  repository: quay.apps.ocp4-8.infocepo.com/<NOM_ORGANISATION>-<NOM_PROJET>/<NOM_REPO>
```
Le nom de l'image est à adapter en fonction des paramètres de création de son espace sur la console DSO, notamment le nom de son organisation, le nom du projet DSO et le nom du repo synchronisé.

le nom complet peut également se trouver depuis gitlab-ci sur la pipeline de construction du projet de tuto java de la branche master sur la dernière étape de build  nommée docker-build :

par exemple 

```log
INFO[0008] Pushing image to quay.apps.ocp4-8.infocepo.com/ministere-interieur-plec/app-java-forge-dso-demo:master 
```

Le deuxième paramètre à modifier est l'URL d'exposition de l'application, qui est précisé dans la partie ingress :

```yaml
ingress:
  enabled: true
  secret:
    enabled: true
  host: tuto.dso.numerique-interieur.com
```

Mettre ici le nom DNS correspondant à son déploiement, de la forme nom_projet.dso.numerique-interieur.com


## Variables
Détails des variables du fichier de déploiement
### Applications

Fichier de variable minimal:

```
image:
  repository: quay.apps.ocp4-8.infocepo.com/ministere-interieur-demo-projets-dso-clients/app-java-forge-dso-demo # Référence de l'image DSO à déployer

ingress:
  enabled: true # Création d'un ingress
  secret:
    enabled: false # Création du secret TLS false si déjà présent
  host: dso-demo-java-helm-plec.apps.ocp4-8.infocepo.com # Nom d'host
  tls:
    - secretName: dso-demo-java-tls # Nom du secret contenant le certificat TLS
``` 

Fichier de variable complet :
```
replicaCount: 1 # Nombre de réplicas du pod

image:
  repository: quay.apps.ocp4-8.infocepo.com/ministere-interieur-demo/app-java-forge-dso-demo # Nom de l'image DSO à déployer, dépendant de l'organisation et du projet DSO créé
  pullPolicy: Always # Policy de récupération de l'image sur le node (Always : à chaque déploiement de pod, IfNotPresent : uniquement sur un node qui ne contient pas l'image en cache)
  tag: "master" # Nom du tag de l'image

imagePullSecrets: 
  - name: "registry-pull-secret" # Nom du secret permettant de se connecter à la registry Quay. Ne pas modifier cette valeur dans le cadre de DSO

service:
  type: ClusterIP # Type de service kubernetes à créer, ClusterIP dans la majorité des cas
  port: 8080 # Port d'exposition du service kubernetes

ingress:
  enabled: true # Création d'un ingress
  secret:
    enabled: false # Création du secret TLS false si déjà présent
  host: dso-demo-java-helm-plec.apps.ocp4-8.infocepo.com # Nom d'host
  tls:
    - secretName: dso-demo-java-tls # Nom du secret contenant le certificat TLS

resources: # Contraintes limit et requests sur les ressources du POD
  limits:
    memory: "512Mi"
    cpu: "500m"  
  requests:
    memory: "512Mi"
    cpu: "500m"  

```

### Base de données
```
# Variable liées au chart bitanmi/postgresql déclarée en dépendance
global: 
  postgresql:
    auth:
      postgresPassword: "MySupErPaSSwOrd" # Mot de passe de l'utilisateur postgreSQL
      username: "userdemo" # Utilisateur en base de données dédié à l'application
      password: "passworddemo" # Mot de passe en base de données dédié à l'application
      database: "demodb" # Nom de la base de données dédiée à l'application
postgresql:
  fullnameOverride: postgres-demo # Nom du service postgresql qui est également utilisé par le deployment applicatif.
# Contrainte de sécurité Openshift
  volumePermissions:
    enabled: false
    securityContext:
      runAsUser: "auto"
  securityContext:
    enabled: false
  shmVolume:
    chmod:
      enabled: false
  containerSecurityContext:
    enabled: false
  podSecurityContext:
    enabled: false
  primary:
    volumePermissions:
      enabled: false
      securityContext:
        runAsUser: "auto"
    securityContext:
      enabled: false
    shmVolume:
      chmod:
        enabled: false
    containerSecurityContext:
      enabled: false
    podSecurityContext:
      enabled: false
```
## Utilisation dans DSO

Une fois le repo ajouté comme repo synchronisé depuis la console DSO, il est nécessaire d'aller sur ArgoCD, depuis la console DSO aller dans le menu *Mes services* puis cliquez sur l'icône ArgoCD, choisir son application et cliquez sur *App Details* cliquez sur le bouton *EDIT* et modifier le champ *PATH* qui est indiqué à *BASE* et le remplacer par . (point)

Problèmes connus :

Les noms de ressources dans Kubernetes doivent faire moins de 63 caractères or DSO génère les noms des ressources à partir du nom de l'organisation et du nom du projet. Il peut arriver que les noms générés fassent plus de 63 caractères et empêchent le déploiement. Depuis Helm, il est possible de surcharger le nom par les 2 varaibles suivantes :
  - nameOverride
  - fullnameOverride

De la même façon pour postgresql :
  - postgresql.nameOverride
  - postgresql.fullnameOverride
