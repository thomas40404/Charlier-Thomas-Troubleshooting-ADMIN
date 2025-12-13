# Exercices de troubleshooting WEB1

- **Auteur** : Charlier Thomas  
- **Date** : 25/11/2025  
- **Usage des IAGs** : Utilisation limitée à la mise en page et à la correction orthographique.

> *GNS3 n’ayant pas accès à Internet, je considère qu’une trace Wireshark vers le routeur équivaut à un accès Internet réussi.*

---

## 1. Bug Report

Je travaille sur un réseau complet contenant :
- un serveur **DHCP** utilisant *dhcpd* ;
- un serveur **DNS résolveur** ;
- un **NS interne** à l’entreprise ;
- un **serveur web** nommé `www` ;
- deux **clients DHCP** : *Direction* et *Atelier*.

### Contexte du problème

L’entreprise **Woodytoys** nous contacte car son serveur web semble capricieux depuis la dernière maintenance, effectuée par un stagiaire.

Le serveur héberge deux sites :
- `www.woodytoys.lab` (site principal)  
- `blog.woodytoys.lab` (blog)

**Problème constaté** :  
L’URL `blog.woodytoys.lab` affiche le site principal au lieu du blog.

Objectif : identifier l’origine du problème et proposer une solution.

---

## 2. Collecte des symptômes

### 2.1 Vérification du DHCP

Commande exécutée sur le PC *Direction* :

```bash
ip a
```

**Résultat** :  
![Img1](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2012-47-02.png?raw=true)

➡️ Le PC reçoit correctement une adresse IP : le serveur DHCP fonctionne.

---

### 2.2 Vérification du DNS

```bash
ping www.woodytoys.lab
```

**Résultat** :  
![Img2](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2012-48-54.png?raw=true)

➡️ La résolution DNS fonctionne correctement.

---

### 2.3 Vérification du site principal

```bash
links http://www.woodytoys.lab
```

**Résultat** :  
![Img3](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2014-54-03.png?raw=true)

➡️ Le site principal est accessible : le serveur web fonctionne.

---

### 2.4 Vérification du blog

```bash
links http://blog.woodytoys.lab
```

**Résultat** :  
![Img4](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2014-54-03.png?raw=true)

➡️ Le site principal s’affiche à la place du blog (comportement incorrect).

---

### 2.5 Vérification des ports du serveur web

Sur le serveur web :

```bash
netstat -nltpu
```

**Résultat** :  
![Img5](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2014-53-17.png?raw=true)

➡️ Les ports **80** et **8000** sont en écoute.  
⚠️ Le port **8000** n’est pas censé être utilisé pour un accès client standard.

---

### 2.6 Test du blog via le port 8000

```bash
links http://blog.woodytoys.lab:8000
```

**Résultat** :  
![Img6](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2014-53-40.png?raw=true)

➡️ Le blog s’affiche correctement **uniquement** en précisant le port 8000.  
❌ Ce n’est pas acceptable pour un utilisateur final.

---

## 3. Outils utilisés

| Outil | Rôle | Résultat attendu |
|------|------|------------------|
| **Wireshark** | Analyse du trafic réseau | Communication correcte client ↔ serveur |
| **ping** | Test de connectivité IP | Réponse du serveur |
| **ip a** | Vérification de l’adresse IP | IP valide dans `192.168.0.0/24` |
| **netstat** | Vérification des ports ouverts | Port 80 (et 443 si HTTPS) |
| **links** | Test d’accès HTTP | Affichage du bon site selon le nom |
| **Logs Apache** | Détection d’erreurs | VirtualHosts correctement associés |

---

## 4. Identification du problème

**Constat** :  
Le blog est accessible uniquement via le port **8000**.

**Hypothèse** :  
Le serveur web utilise une **virtualisation par port**, alors que le HTTP standard utilise le port **80**.  
La configuration correcte doit utiliser une **virtualisation par nom** (*NameVirtualHost*).

---

## 5. Proposition de solution

Modification du fichier de configuration Apache :

```bash
/etc/apache2/sites-available/blog-woodytoys-lab.conf
```

Action effectuée :
- remplacement du port **8000** par le port **80** dans le VirtualHost.

![Img7](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2014-53-40.png?raw=true)

---

## 6. Validation de la solution

```bash
links http://blog.woodytoys.lab
```

**Résultat** :  
![Img8](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/3_TroubleShooting_WEB/Capture%20d%E2%80%99%C3%A9cran%20du%202025-12-13%2015-09-26.png?raw=true)

✅ Le blog s’affiche correctement **sans préciser de port**.

---

## 7. Conclusion

Le problème provenait d’une mauvaise configuration des VirtualHosts Apache utilisant une virtualisation par port.

La correction permet :
- un accès standard sur le port 80 ;
- une meilleure expérience utilisateur ;
- une configuration conforme aux bonnes pratiques HTTP.