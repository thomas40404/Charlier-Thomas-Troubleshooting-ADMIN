
# < Exercices de troubleshooting DHCP 2 >

- Auteur(s) :  Charlier Thomas
- Date : 08/11/2025


## 1. Bug Report

# < Exercices de troubleshooting DHCP 2 >

- Auteur(s) :  Charlier Thomas
- Date : 08/11/2025


## 1. Bug Report
Je suis dans un réseau complet contenant un serveur DHCP et un serveur DNS, ainsi que deux clients DHCP
Les utilisateurs me disent que quelqu'un est venu faire des modifications et depuis seulement un client DHCP n'arrive a se connecter a internet a la fois puis parfois le poste pouvant le faire change. 
Avant cela tous les clients DHCP pouvaient aller sur internet simultanément.
Le serveur DHCP utilise DHCPD pour fournir des IP aux clients.

 
## 2. Collecte des symptômes
Je vais commencer à faire un ping 1.1.1.1 avec le PC Direction, je vois que celui-ci n’est pas capable de ping.
Je fais donc un ip a et vois qu’il n’a pas d’IP.
Je m’empresse de faire de même sur Atelier et vois que le ping passe et que l’IP est 192.168.0.10.
Je reviens plus tard voir, car mon client me dit que cela change parfois de PC après un certain temps.
Je vais réessayer avec Direction, et oui, lui peut maintenant se connecter. Je vérifie son IP et vois que son IP est aussi 192.168.0.10.
Normalement, les deux pourraient le faire en même temps et leurs IP seraient différentes.
Je fais donc une capture Wireshark et vois que les PC sans IP font des Discover sans réponses.
Parfois, les deux récupèrent l’IP .10 au même moment mais sont incapables de se connecter.


### Liste des outils :

- Wireshark,
- ping, 
- ip a, 
- dhcpd -d


## 3. Identification et description du problème  
   * Mon “bug” est le fait que seulement un client DHCP ne peut se connecter à Internet.
   * J’investigue et récolte un indice : tous les clients DHCP fonctionnels ont l’IP 192.168.0.10, les autres font des Discover à l’infini.
   * Hypothèse : la configuration DHCP a été modifiée et seule l’IP .10 est disponible. Le premier arrivé sur le réseau est donc le premier servi jusqu’à la fin de son bail DHCP.

Je pense que le fichier DHCPD ne permet que de récupérer l’IP .10. 

## 4. Proposition de solution  
Je vais dans la configuration DHCPD : /etc/dhcp/dhcpd.conf.
Je constate que le range est de 192.168.0.10 à 192.168.0.10. Je modifie donc pour 192.168.0.10 à 192.168.0.100.
Cela va permettre au serveur de donner 90 IP différentes. Ainsi, maintenant, 90 PC configurés en DHCP pourront être sur ce réseau interne.

nano /etc/dhcp/dhcpd.conf 
modification de la ligne 
      range 192.168.0.10 19.168.0.10; 
   par 
      range 192.168.0.10 19.168.0.100;


