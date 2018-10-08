# STEP 2
## OBJECTIF :

L'équipe de développement en charge de la migration du site web de l'entreprise à besoin d'un environnement sur lequel elle pourra déployer et tester le nouveau site web en .Net Core.

## SOLUTION :

### **Fichier template.json**

#### _App Service Plan_

1- saisir __*"arm-plan"*__ pour ajouter la ressource de type *serverfarms*

![](/assets/S2-ArmPlan.png "Picture 1")

#### *web App*

2- ajouter un paramètre pour le nom unique de la web app 

```
"webappName": {
   "type": "string",
   "metadata": {
        "description": "Nom personnalisé"
    }
}
```

3- saisir __*"arm-webapp"*__ pour ajouter la ressource de type *sites*

- remplacer **WEB_APP_NAME** par *__[parameters('webappName')]__*
- remplacer **APP_SERVICE_PLAN_NAME** par *__planApp__*

```
{
    "apiVersion": "2015-08-01",
    "name": "[parameters('webappName')]",
    "type": "Microsoft.Web/sites",
    "location": "[resourceGroup().location]",
    "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/appPlan')]": "Resource",
        "displayName": "[parameters('webappName')]"
    },
    "dependsOn": [
        "Microsoft.Web/serverfarms/appPlan"
    ],
    "properties": {
        "name": "[parameters('webappName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms/', 'appPlan')]"
    }
}
```

### **Fichier template.parameters.json**

4- renseigner la valeur pour le paramètre **webappName** (*pas celui du voisin*)

```
"webappName": {
    "value": "..."
}
```

### **Déploiement via Powershell**

Suivre la procédure de déploiement du Step 1 avec Powershell.