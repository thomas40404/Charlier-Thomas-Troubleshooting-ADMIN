# Exercices de troubleshooting DNS1

- **Auteur(s)** : Charlier Thomas  
- **Date** : 25/11/2025  
- **Usage des IAGs** : Utilisation limitée à la mise en page et à la correction orthographique.

*GNS3 n’ayant pas accès à Internet, je considérerai qu’une trace Wireshark vers le routeur est un accès Internet réussi.*
---

## 1. Bug Report

Je travaille sur un réseau complet contenant un serveur DHCP utilisant *dhcpd*, un serveur DNS résolveur, un NS interne à l’entreprise, ainsi que deux clients DHCP : Direction et Atelier.

Le client me dit avoir des problèmes de connexion lorsqu’il est connecté dans l’entreprise.

---

## 2. Collecte des symptômes

Depuis le PC Direction, un ping vers 1.1.1.1 réussit car une trace vers le routeur est reçue (rappel : la VM n’a pas d’accès à Internet, un ping ne peut donc pas réellement réussir).

Depuis le PC Direction, je ping **google.com** : ma trace vers le résolveur est bien reçue, mais le résolveur répond « Refused ».  
![capture](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/2_TroubleShooting_DNS/capture_refused.png?raw=true)

Je regarde l’IP du PC Direction et je vois : **192.168.0.10**.

### Liste des outils utilisés et leur rôle

1. **Wireshark**  
   - **Rôle** : Analyser le trafic réseau entre le client et le serveur DNS.  
   - **Valeur attendue** : Requêtes DNS acceptées et réponses reçues.

2. **ping**  
   - **Rôle** : Tester la connectivité IP.  
   - **Valeur attendue** : Réponse du serveur cible.

3. **ip a**  
   - **Rôle** : Vérifier l’adresse IP assignée au PC.  
   - **Valeur attendue** : IP dans le réseau correct (192.168.0.0/24).

4. **netstat**  
   - **Rôle** : Vérifier les ports et connexions actives sur le serveur DNS.  
   - **Valeur attendue** : Port DNS à l’écoute et acceptant les requêtes du réseau interne.

5. **named -g**  
   - **Rôle** : Lancer le serveur DNS en foreground pour observer les logs.  
   - **Valeur attendue** : Requêtes DNS acceptées pour les IP autorisées.

---

## 3. Identification et description du problème

Le résolveur DNS refuse les requêtes provenant du réseau réel (**192.168.0.0/24**) car la configuration indique :

```
allow-recursion {
    192.168.1.0/24;  --> tous les 192.168.1.*
    127.0.0.1/32;    --> pour lui-même
};
```

**Hypothèse** : Les clients ont une IP dans 192.168.0.0/24 mais ne sont pas inclus dans la plage autorisée par le serveur DNS.

---

## 4. Proposition de solution

Modification de la configuration du serveur DNS dans `/etc/bind/named.conf` :

```
allow-recursion {
    192.168.0.0/24;  --> tous les 192.168.0.*
    127.0.0.1/32;    --> pour lui-même
};
```

**Validation** :  
- Ping vers le routeur réussi (trace reçue)  
- Ping vers `www.woodytoys.lab` fonctionne  
- Le serveur accepte désormais les requêtes DNS des clients  

![config avant résolution](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/2_TroubleShooting_DNS/config1.png?raw=true)  
![trace après résolution](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/2_TroubleShooting_DNS/traceFIn.png?raw=true)  
![ping final](https://github.com/thomas40404/Charlier-Thomas-Troubleshooting-ADMIN/blob/main/2_TroubleShooting_DNS/ping%20.png?raw=true)

Cette solution limite les IP autorisées au réseau réel, ce qui est plus sûr.

---
