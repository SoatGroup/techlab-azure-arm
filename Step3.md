# STEP 3
## OBJECTIF :

Avec les identifiants, l'équipe en charge du service web WCF ont pu mener à bien son intégration sur la machine virtuelle. Maintenant l'équipe vous demande de limiter le RDP aux personnes du réseau d'entreprise (*62.62.62.1*) et d'autoriser le port 8090. Ils voudraient également un nouveau subnet "Data" pour la prochaine machine virtuelle qui va contenir un server SQL.

L'équipe de développement vous demande également de mettre à disposition le nom du compte de stockage ainsi qu'une clé d'accès, dans l'_appSettings_ de la web app.

## INDICES :

### **Fichier template.json**

#### **Parameters**

1- créer un nouveau paramètre __*subnets*__ avec les informations des deux sous réseaux :

```
"subnets":{
    "type": "array",
    "defaultValue": [
        {
            "Name": "Web",
            "Prefix": "10.0.1.0/24"
        },
        {
            "Name": "Data",
            "Prefix": "10.0.2.0/24"
        }
    ],
    "metadata": {
        "description" : "List of subnets to create"
    }
}
```

#### **Variables**

2- ajouter une variable qui fait référence au paramètre ci-dessus :

```
"subnets": "[parameters('subnets')]"
```

3- supprimer la variable **subnetPrefix**

#### **Resources**

4- dans la ressource **Microsoft.Network/virtualNetworks**, remplacer la propriété **subnets** par :

```
"copy": [
    {
        "name": "subnets",
        "count": "[length(variables('subnets'))]",
        "input": {
            "name": "[variables('subnets')[copyIndex('subnets')].Name]",
            "properties": {
                "addressPrefix": "[variables('subnets')[copyIndex('subnets')].Prefix]"
            }
        }
    }
]
```

5- dans la ressource **Microsoft.Network/networkSecurityGroups**, modifier la règle de sécurité **"rdp_rule"**, remplacer la valeur de la propriété **"sourcePortRange"** par **_62.62.62.1/32_**

```
    "name": "rdp_rule",
    "properties": {
      "description": "Restricted RDP access",
      "protocol": "Tcp",
      "sourcePortRange": "*",
      "destinationPortRange": "3389",
      "sourceAddressPrefix": "62.62.62.1/32",
      "destinationAddressPrefix": "*",
      "access": "Allow",
      "priority": 100,
      "direction": "Inbound"
    }
```

6- ajouter une nouvelle règle de sécurité (*arm-nsgrule*)

```
    "name": "http_8090_rule",
    "properties": {
        "description": "Http 8090 port allowed",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "destinationPortRange": "8090",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "*",
        "access": "Allow",
        "priority": 101,
        "direction": "Inbound"
    }
```

7- Dans la ressource **Microsoft.Web/sites**, ajouter les deux configurations relatives au compte de stockage

```
"siteConfig": {
    "appSettings": [
        {
            "Name": "STORAGEACCOUNT_NAME",
            "Value": "[variables('storageName')]"
        },
        {
            "Name": "STORAGEACCOUNT_PRIMARY_KEY",
            "Value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageName')), '2015-06-15').key1]"
        }
    ]
}
```

### **Déploiement via Powershell**

Suivre la procédure de déploiement du Step 1 avec Powershell.