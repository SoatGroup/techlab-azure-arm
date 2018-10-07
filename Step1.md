# STEP 1
## OBJECTIF :

Votre entreprise a besoin d'un environnement pour installer un service web legacy en WCF. Elle vous demande donc de leur mettre à disposition, sur le portail Azure, une machine virtuelle Windows Server 2012 avec IIS et la possibilité de s'y connecter en RDP.

## SOLUTION :
### **Fichier template.json**

1- créer un fichier **"template.json"** à la racine du dossier

![](assets\3-.jpg "Picture 1")

4- ajouter le squelette du template ARM dans le fichier JSON en tapant **_arm!_** et valider

5- cliquer entre **"[]"** de la propriété **"resources"** et saisir __*arm-vm-windows*__ et valider

![](assets\5-.jpg "Picture 4")

6- remplacer le mot en surbrillance **WindowsVM1** par __*demovm*__ 

7- rechercher l'extension de la machine virtuelle  **"demovmAzureDiagnostics"** et remplacer la par celle qui va nous permettre de configurer **IIS** :

```
{
  "type": "extensions",
  "name": "demovmDSCExtension",
  "apiVersion": "2015-05-01-preview",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "Microsoft.Compute/virtualMachines/demovm"
  ],
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.19",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "configuration": {
        "url": "https://contenttechlab.blob.core.windows.net/contenu/DemoARM.zip",
        "script": "[parameters('moduleName')]",
        "function": "[parameters('configurationFunction')]"
      },
      "configurationArguments": {
        "MachineName": "demovm"
      }
    },
    "protectedSettings": null
  }
}
```

8- ajouter une ressource après la machine virtuelle, taper __*arm-nsg*__ puis valider

9- remplacer le mot en surbrillance par __*demovmNsg*__

10- ajouter la règle de sécurité suivante (il est possible d'utiliser __*"arm-nsgrule"*__)

```
"securityRules": [
  {
    "name": "rdp_rule",
    "properties": {
      "description": "Open bar RDP",
      "protocol": "Tcp",
      "sourcePortRange": "*",
      "destinationPortRange": "3389",
      "sourceAddressPrefix": "Internet",
      "destinationAddressPrefix": "*",
      "access": "Allow",
      "priority": 100,
      "direction": "Inbound"
    }
  }
]
```

11- rechercher la ressource __"Microsoft.Network/networkInterfaces"__ dans ses propriétés à la suite de __"ipConfigurations"__ ajouter

 ```
"networkSecurityGroup": {
  "id": "[resourceId('Microsoft.Network/networkSecurityGroups', 'demovmNsg')]"
}
 ```

####	*PARAMETERS*
12- pour ajouter un paramètre, cliquer entre **"{}"** de la propriété **"parameters"** et saisir __*arm-parameter*__ et valider

![](assets\7-.jpg "Picture 5")

13- remplacer **"parameterName"** par __*"dnsNameForPublicIP"*__ et ajouter la description __*"Nom unique du DNS pour l'IP publique"*__

 ```
"dnsNameForPublicIP": {
    "type": "string",
    "metadata": {
        "description": "Nom unique du DNS pour l'IP publique"
    }
}
 ```

14- ajouter un autre paramètre __*"adminUserName"*__ avec la description __*"Login du compte admin de la VM"*__

15- ajouter un autre paramètre __*"adminPassword"*__ de type __"securestring"__ avec la description __*"Mot de passe du compte admin de la VM"*__

```
"adminUserName": {
   "type": "string",
   "metadata": {
        "description": "Login du compte admin de la VM"
    }
},
"adminPassword": {
   "type": "securestring",
   "metadata": {
        "description": "Mot de passe du compte admin de la VM"
    }
}
```

16- rechercher la propriété __domainNameLabel__ et remplacer sa **valeur** par __*[parameters('dnsNameForPublicIP')]*__

17- chercher __ADMIN_USERNAME__ et remplacer par __*[parameters('adminUserName')]*__

18- chercher __ADMIN_PASSWORD__ et remplacer par __*[parameters('adminPassword')]*__

19- ajouter le paramètre __*"windowsOSVersion"*__ avec la description __*"Version de Windows pour la VM"*__ :

- a- lui ajouter une propriété __*"allowedValues"*__ avec comme valeur le tableau 
		__*["2012-R2-Datacenter-smalldisk", "2012-R2-Datacenter", "2008-R2-SP1"]*__
- b- lui ajouter une propriété __*"defaultValue"*__ avec la valeur __*"2012-R2-Datacenter-smalldisk"*__

```
"windowsOSVersion": {
   "type": "string",
   "defaultValue": "2012-R2-Datacenter-smalldisk",
   "allowedValues": [
       "2008-R2-SP1",
       "2012-R2-Datacenter",
       "2012-R2-Datacenter-smalldisk"
   ],
   "metadata": {
        "description": "Version de Windows pour la VM"
    }
}
```

20- rechercher la propriété __"sku"__ et remplacer sa __valeur__ par __*"[parameters('windowsOSVersion')]"*__

21- ajouter les deux paramètres de l'extension **moduleName** et **configurationFunction** de type _**string**_

```
"moduleName": {
  "type": "string",
  "metadata": {
    "description": "URL for the DSC configuration module.
  }
},
"configurationFunction": {
  "type": "string",
  "defaultValue": "DemoARM.ps1\\DemoARM",
  "metadata": {
    "description": "DSC configuration function to call"
  }
}
```

#### *VARIABLES*
22- ajouter deux variables via la commande __"arm-variable"__ :

- __*"addressPrefix":"10.0.0.0/16"*__
- __*"subnetPrefix":"10.0.0.0/24"*__

23- dans la ressource __"demovm-VirtualNetwork"__ remplacer les valeurs des adresses Ip par les variables :

- __*[variables('addressPrefix')]*__
- __*[variables('subnetPrefix')]*__

Il est possible d'utiliser les variables pour remplacer les identifiants trop long, exemple avec l'id du subnet dans la ressource __"demovm-NetworkInterface"__. 

24- Pour cela créer deux variables :

- __"vnetId"__ avec la valeur __*"[resourceId('Microsoft.Network/virtualNetworks', 'demovm-VirtualNetwork')]"*__
- __"subnetRef"__ avec la valeur __*"[concat(variables('vnetId'), '/subnets/demovm-VirtualNetwork-Subnet')]"*__

25- remplacer la valeur de l'id du subnet par __*"[variables('subnetRef')]"*__

![](assets\20-.jpg "Picture 10")

#### *TAG*
26- ajouter un tag supplémentaire à la ressource de la machine virtuelle __*"BusinessUnit": "FINANCE"*__ pour la gestion des coûts

### **Fichier template.parameters.json**

27- pour générer ce fichier faite un clique droit sur le fichier template.json et sélectionner :

![](assets\22-.jpg "Picture 11")

28- renseigner les valeurs comme ci-dessous :

- **"dnsNameForPublicIP"**  :  __*"demovmdns"*__
- **"adminUserName"**  :  __*"adminDemo"*__
- **"adminPassword"**  :  __*"P@ssw0rd#2018"*__
- **"moduleName"**  :  __*"DemoARM.ps1"*__
- **"configurationFunction"** : __*"Main"*__

*pour le premier paramètre ajouter un nom de domaine personalisé*

### **Déploiement via Powershell**

29- afficher le terminal à l'aide du raccourci clavier **Ctrl+Shift+ù**

30- lancer le déploiement avec la commande suivante : 

```
New-AzureRmResourceGroupDeployment -Name ARMDeployment -ResourceGroupName demoARM -TemplateFile template.json -TemplateParameterFile template.parameters.json
```

*Résultat du déploiement :*

![](assets\S1-28-.jpg "Picture 12")

### **Fichier template.json**

31- ajouter **"Identity"** à la ressource machine virtuelle

![](assets\S1-32-.jpg "Picture 13")

### **Déploiement via Powershell**

32- relancer le déploiement avec la commande suivante : 

```
New-AzureRmResourceGroupDeployment -Name ARMDeployment -ResourceGroupName demoARM -TemplateFile template.json -TemplateParameterFile template.parameters.json
```

*Résultat du déploiement :*

![](assets\S1-33-.jpg "Picture 13")