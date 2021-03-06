# STEP 1
## OBJECTIF :

Votre entreprise a besoin d'un environnement pour installer un service web legacy en WCF. Elle vous demande donc de leur mettre à disposition, sur le portail Azure, une machine virtuelle Windows Server 2012 avec IIS et la possibilité de s'y connecter en RDP.

## SOLUTION :
### **Fichier template.json**

1- créer un fichier **"template.json"** à la racine du dossier

![](/assets/S1-File.png "Picture 1")

2- ajouter le squelette du template ARM dans le fichier JSON en tapant **_arm!_** et valider

3- cliquer entre **"[]"** de la propriété **"resources"** et saisir __*arm-vm-windows*__ et valider

![](/assets/S1-ArmVmWindows.png "Picture 4")

4- remplacer le mot en surbrillance **WindowsVM1** par __*demovm*__ 

5- rechercher l'extension de la machine virtuelle  **"demovmAzureDiagnostics"** et remplacer la par celle qui va nous permettre de configurer **IIS** :

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
        "url": "https://raw.githubusercontent.com/SoatGroup/techlab-azure-arm/master/DSC/DemoARM.zip",
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

6- ajouter une ressource après la machine virtuelle, taper __*arm-nsg*__ puis valider

7- remplacer le mot en surbrillance par __*demovmNsg*__

8- ajouter la règle de sécurité suivante (il est possible d'utiliser __*"arm-nsgrule"*__)

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

9- rechercher la ressource __"Microsoft.Network/networkInterfaces"__ dans ses propriétés à la suite de __"ipConfigurations"__ ajouter

 ```
"networkSecurityGroup": {
  "id": "[resourceId('Microsoft.Network/networkSecurityGroups', 'demovmNsg')]"
}
 ```

####	*PARAMETERS*
10- pour ajouter un paramètre, cliquer entre **"{}"** de la propriété **"parameters"** et saisir __*arm-parameter*__ et valider

11- remplacer **"parameterName"** par __*"dnsNameForPublicIP"*__ et ajouter la description __*"Nom unique du DNS pour l'IP publique"*__

 ```
"dnsNameForPublicIP": {
    "type": "string",
    "metadata": {
        "description": "Nom unique du DNS pour l'IP publique"
    }
}
 ```

12- ajouter un autre paramètre __*"adminUserName"*__ avec la description __*"Login du compte admin de la VM"*__

13- ajouter un autre paramètre __*"adminPassword"*__ de type __"securestring"__ avec la description __*"Mot de passe du compte admin de la VM"*__

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

14- rechercher la propriété __domainNameLabel__ et remplacer sa **valeur** par __*[parameters('dnsNameForPublicIP')]*__

15- chercher __ADMIN_USERNAME__ et remplacer par __*[parameters('adminUserName')]*__

16- chercher __ADMIN_PASSWORD__ et remplacer par __*[parameters('adminPassword')]*__

17- ajouter le paramètre __*"windowsOSVersion"*__ avec la description __*"Version de Windows pour la VM"*__ :

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

18- rechercher la propriété __"sku"__ et remplacer sa __valeur__ par __*"[parameters('windowsOSVersion')]"*__

19- ajouter les deux paramètres de l'extension **moduleName** et **configurationFunction** de type _**string**_

```
"moduleName": {
  "type": "string",
  "metadata": {
    "description": "URL for the DSC configuration module."
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
20- ajouter deux variables via la commande __"arm-variable"__ :

- __*"addressPrefix":"10.0.0.0/16"*__
- __*"subnetPrefix":"10.0.1.0/24"*__

21- dans la ressource __"demovm-VirtualNetwork"__ remplacer le nom du subnet par __*Web*__ et les valeurs des adresses Ip par les variables :

- __*[variables('addressPrefix')]*__
- __*[variables('subnetPrefix')]*__

Il est possible d'utiliser les variables pour remplacer les identifiants trop long, exemple avec l'id du subnet dans la ressource __"demovm-NetworkInterface"__. 

22- Pour cela créer deux variables :

- __"vnetId"__ avec la valeur __*"[resourceId('Microsoft.Network/virtualNetworks', 'demovm-VirtualNetwork')]"*__
- __"subnetRef"__ avec la valeur __*"[concat(variables('vnetId'), '/subnets/Web')]"*__

23- remplacer la valeur de l'id du subnet par __*"[variables('subnetRef')]"*__

![](/assets/S1-SubnetRef.png "Picture 10")

24- Ajouter une variable pour le nom du Storage Account
- __"storageName": "[uniqueString(resourceGroup().id)]"__

25- Remplacer les occurences du nom du storage **toLower('demovmstorage')** par la variable précédemment créer
- Dans le nom de la ressource Storage
- Dans l'objet dependsOn de la machine virtuelle
- Dans l'objet bootDiagnostics de la machine virtuelle


#### *TAG*
26- ajouter un tag supplémentaire à la ressource de la machine virtuelle __*"BusinessUnit": "FINANCE"*__ pour la gestion des coûts

### **Fichier template.parameters.json**

27- pour générer ce fichier faite un clique droit sur le fichier template.json et sélectionner :

![](/assets/S1-ParametersFile.png "Picture 11")

28- renseigner les valeurs comme ci-dessous :

- **"dnsNameForPublicIP"**  :  __*"demovmdns"*__ (doit être unique, donc pas celui du voisin)
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

![](/assets/S1-ResultatEchec.png "Picture 12")

### **Fichier template.json**

31- ajouter **"Identity"** à la ressource machine virtuelle

![](/assets/S1-IdentityVm.png "Picture 13")

### **Déploiement via Powershell**

32- relancer le déploiement avec la commande suivante : 

```
New-AzureRmResourceGroupDeployment -Name ARMDeployment -ResourceGroupName demoARM -TemplateFile template.json -TemplateParameterFile template.parameters.json
```

33- aller sur le portail Azure et surveiller le déploiement jusqu'à l'extension. 

![](/assets/S1-AzDeploy.png "Picture 14")

à ce stade vous pouvez passer au Step2-a car le déploiement dure environ 20 min.

*Résultat du déploiement :*

![](/assets/S1-Resultat.png "Picture 13")