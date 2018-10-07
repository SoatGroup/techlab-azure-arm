# STEP 0
## OBJECTIF :

Ajouter une Policy concernant les prochaines créations de machine virtuelle qui devront avoir obligatoirement une Identity de configuré. On profitera également de ce déploiement pour créer un groupe de ressource que nous utiliserons pour les prochains déploiements.

## SOLUTION :
### **Fichier identity_policy.json**

1- création d'un dossier __*"TechLab"*__ pour le lab

![](/assets/S0-Folder.png "Picture 1")

2- ouvrir ce dossier dans Visual Code

3- créer un fichier **"identity_policy.json"** à la racine du dossier

![](/assets/S0-File.png "Picture 2")

4- éditer le fichier JSON en tapant **"arm!"** et valider

![](/assets/S0-Arm.png "Picture 3")

5- cliquer entre **"[]"** de la propriété **"resources"** et copier la définition de la **Policy**

```
{
    "name": "vm-creation-need-identity-definition",
    "type": "Microsoft.Authorization/policyDefinitions",
    "apiVersion": "2018-03-01",
    "properties": {
        "displayName" : "Deny vm without Identity",
        "policyType": "Custom",
        "mode": "All",
        "description" : "Deny vm without Identity",
         "policyRule": {
            "if": {
                "allOf": [
                  {
                    "field": "type",
                    "equals": "Microsoft.Compute/virtualMachines"
                  },
                  {
                    "not": {
                      "field": "identity.type",
                      "equals": "SystemAssigned"
                    }
                  }
                ]
              },
              "then": {
                "effect": "deny"
              }
        }
    }
}
```

6- toujours dans la propriété **"resources"** copier l'assignation de la **Policy** sur la souscription

```
{
    "name": "vm-creation-need-identity-assignment",
    "type": "Microsoft.Authorization/policyAssignments",
    "apiVersion": "2018-03-01",
    "dependsOn": [
        "[resourceId('Microsoft.Authorization/policyDefinitions/', 'vm-creation-need-identity-definition')]"
    ],
    "properties": {
        "displayName" : "Use denied VM without Identity",
        "description" : "Use denied VM without Identity",
        "metadata" : {
            "assignedBy" : "Admin"
        },
        "scope": "[subscription().id]",
        "policyDefinitionId": "[resourceId('Microsoft.Authorization/policyDefinitions', 'vm-creation-need-identity-definition')]"
    }
}
```

7- Ajouter également la définition du groupe de ressource

```
{
    "type": "Microsoft.Resources/resourceGroups",
    "apiVersion": "2018-05-01",
    "location": "[parameters('rgLocation')]",
    "name": "[parameters('rgName')]",
    "properties": {}
}
```

####	*PARAMETERS*
8- cliquer entre **"{}"** de la propriété **"parameters"** et saisir __*"arm-parameter"*__ et valider

![](/assets/S0-Parameter.png "Picture 4")

9- remplacer **"parameterName"** par __*"rgLocation"*__ 

10- ajouter un autre paramètre __*"rgName"*__

```
"rgLocation": {
    "type": "string"
},
"rgName": {
    "type": "string"
}
```

### **Déploiement via Powershell**

11- afficher le terminal à l'aide du raccourci clavier **Ctrl+Shift+ù**

12- se connecter au portail Azure : 

```
Connect-AzureRmAccount
```

13- pour sélectionner une souscription : 

```
Select-AzureRmSubscription -SubscriptionName SoCloudSandbox
```

14- lancer le déploiement avec la commande suivante : 

```
New-AzureRMDeployment -Name demoPolicyAndRG -Location westeurope -TemplateFile identity_policy.json -rgName demoARM -rgLocation westeurope
```
*Résultat du déploiement :*

![](/assets/S0-Resultat.png "Picture 5")

