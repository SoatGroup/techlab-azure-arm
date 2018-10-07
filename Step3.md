# STEP 3
## OBJECTIF :

Avec les identifiants, l'équipe en charge du service web WCF ont pu mener à bien son intégration sur la machine virtuelle. Maintenant l'équipe vous demande de limiter le RDP aux personnes du réseau d'entreprise (*62.62.62.1*) et d'autoriser le port 8090. L'équipe de développement vous demande également de mettre à disposition le nom du compte de stockage ainsi qu'une clé d'accès, dans l'_appSettings_ de la web app.

## INDICES :

### **Fichier template.json**

1- dans la ressource **Microsoft.Network/networkSecurityGroups** remplacer la valeur de **"sourcePortRange"** par **_62.62.62.1/32_**

2- ajouter la nouvelle règle suivante

```

```

### **Fichier template.parameters.json**