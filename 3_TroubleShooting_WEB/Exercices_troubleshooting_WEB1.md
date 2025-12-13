# Exercices de troubleshooting WEB1

- **Auteur(s)** : Charlier Thomas  
- **Date** : 25/11/2025  
- **Usage des IAGs** : Utilisation limitée à la mise en page et à la correction orthographique.

*GNS3 n’ayant pas accès à Internet, je considérerai qu’une trace Wireshark vers le routeur est un accès Internet réussi.*

---

## 1. Bug Report

Je travaille sur un réseau complet contenant un serveur DHCP utilisant *dhcpd*, un serveur DNS résolveur, un NS interne à l’entreprise,un server web nommé "www" ainsi que deux clients DHCP : Direction et Atelier.

La situation est la suivante :  

L'entreprise Woodytoys vous contacte car son serveur web semble capricieux depuis la dernière maintenance, effectuée par un stagiaire.  Le serveur gère deux sites : le site www.woodytoys.lab et le site blog.woodytoys.lab.  Actuellement, l'adresse blog.woodytoys.lab affiche le site web principal au lieu du blog lui-même! . Pourriez-vous jeter un coup d'œil afin de trouver une solution au problème?

---

## 2. Collecte des symptômes

Pour commencer je vérifie le démarrage de tous les serveurs qui semblent fonctionner correctement, pour ce faire : 
Je fais ```ip a```sur le pc directeur.
Resutlat : 
[Img1]()
Ce qui me dir que le serveur DHCP fonctionne bien 

Ensuite je fais ```ping www.woodytoys.lab```  
Resutlat : 
[Img2]()
Ce qui me dit que les serveurs DNS fonctionnent 

Je fais ```links http://www.woodytosy.lab``` 
Resutlat : Je me retrouve sur le site www.woodytosy.lab
[Img3]()
Ce qui me dit que le serveur web fonctionne

Je fais ```links http://blog.woodytosy.lab``` 
Resutlat : Je me retrouve aussi sur le site www.woodytosy.lab (problème que le client m'avait dit)
[Img4]()
Ce qui n'est pas attendu

Je vais sur le serveur mail et je fais ```netstat -nltpu````
Resutlat : les ports 80 et 8000 sont en écoute 
[Img5]()
Le port 8000 n'est pas censé être en écoute

Je fais donc ```links http://blog.woodytosy.lab:8000``` 
Resutlat : Je me retrouve sur le site log.woodytosy.lab 
[Img6]()
Parfait me voila enfin sur le site voulu.



### Liste des outils utilisés et leur rôle

1. **Wireshark**  
   - **Rôle** : Analyser le trafic réseau entre le client et le serveur WEB.  
   - **Valeur attendue** : 

2. **ping**  
   - **Rôle** : Tester la connectivité IP.  
   - **Valeur attendue** : Réponse du serveur cible.

3. **ip a**  
   - **Rôle** : Vérifier l’adresse IP assignée au PC.  
   - **Valeur attendue** : IP dans le réseau correct (192.168.0.0/24).

4. **netstat - nltp(u)**  
   - **Rôle** : Vérifier les ports et connexions actives sur un serveur.  
   - **Valeur attendue** : Port 80 (http) et potentiellement 443 (https) à l’écoute et acceptant les requêtes du réseau interne.

6. **links**  
   - **Rôle** : Tester l’accès aux sites web en mode texte directement depuis le serveur ou un client, sans interface graphique.  
   - **Valeur attendue** :  
     - `www.woodytoys.lab` → affichage du site principal  
     - `blog.woodytoys.lab` → affichage du blog (VirtualHost correct)

8. **logs Apache/Nginx**  
   - **Rôle** : Identifier les erreurs de configuration du serveur web (VirtualHosts, noms de domaine).  
   - **Valeur attendue** :  
     - Absence d’erreurs critiques  
     - Les requêtes vers `blog.woodytoys.lab` sont traitées par le bon VirtualHost

---

## 3. Identification et description du problème

**Explication** : Je me retrouve sur le bon site en précisant le port 8000 

**Hypothèse** : Le serveur web utilise la virtualisation par port mais une requète http utilise normalement le port 80 et non 8000, il faudrait donc utiliser la virtualisation par nom plutot que par port pour pouvoir utiliser le meme port pour tous (le 80)
---

## 4. Proposition de solution
Je vais dans /etc/apache2/sitesavailable/blog-woodytoys-lab.conf 
Je change 8000 par 80 
[Img7]()


**Validation** :  
Je fais ```links http://blog.woodytosy.lab``` 
Resutlat : Je me retrouve maintenant sur le site blog.woodytosy.lab sans préciser le port 8000
[Img8]()

---