# STEP 3
## OBJECTIF :

Avec les identifiants, l'équipe en charge du service web WCF ont pu mener à bien son intégration sur la machine virtuelle. Maintenant l'équipe vous demande de limiter le RDP aux personnes du réseau d'entreprise (*62.62.62.1*) et d'autoriser le port 8090. L'équipe de développement vous demande également de mettre à disposition le nom du compte de stockage ainsi qu'une clé d'accès, dans l'_appSettings_ de la web app.

## INDICES :

### **Fichier template.json**

1- dans la ressource **Microsoft.Network/networkSecurityGroups**, modifier la règle de sécurité **"rdp_rule"**, remplacer la valeur de la propriété **"sourcePortRange"** par **_62.62.62.1/32_**

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

2- ajouter une nouvelle règle de sécurité (*arm-nsgrule*)

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

3- Dans la ressource **Microsoft.Web/sites**, ajouter les deux configurations relatives au compte de stockage

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