# STEP 2
## OBJECTIF :

L'équipe de développement en charge de la migration du site web de l'entreprise à besoin d'un environnement sur lequel elle pourra déployer et tester le nouveau site web en .Net Core.

## SOLUTION :

### **Fichier template.json**

#### _App Service Plan_

1- saisir __*"arm-plan"*__ pour ajouter la ressource de type *serverfarms*

![](https://github.com/SoatGroup/techlab-azure-arm/blob/master/assets/S2-1-.jpg "Picture 1")

#### *web App*

2- saisir __*"arm-webapp"*__ pour ajouter la ressource de type *sites*

- remplacer **WEB_APP_NAME** par *__webApp__*
- remplacer **APP_SERVICE_PLAN_NAME** par *__planApp__*

![](https://github.com/SoatGroup/techlab-azure-arm/blob/master/assets/S2-2-.jpg "Picture 2")

### **Déploiement via Powershell**

Suivre la procédure de déploiement du Step 1 avec Powershell.