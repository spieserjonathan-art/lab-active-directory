# lab-active-directory
Introduction
L'objectif de cette expérimentation est de mettre en place une infrastructure réseau complète sous Debian à l'aide de machines virtuelles VirtualBox.
Le serveur central nommé ServEntreprise fournit plusieurs services réseau indispensables :
•	DHCP 
•	DNS 
•	FTP 
•	HTTP 
Deux postes clients récupèrent automatiquement leur configuration réseau grâce au protocole DHCP(Debianclient1 et Debianclient2).
Les différents échanges réseau sont analysés à l'aide du logiciel Wireshark afin d'observer le fonctionnement réel des protocoles étudiés.
________________________________________
Présentation de l'architecture
Réseau utilisé
192.168.50.0/24
Machines virtuelles
Machine	Nom	Adresse IP
Serveur	ServEntreprise	192.168.50.100
Client 1	Debianclient1	192.168.50.35
Client 2	Debianclient2	192.168.50.43
Configuration VirtualBox
Les trois machines utilisent :
Mode : Réseau interne
Nom du réseau : serventreprise
________________________________________
Configuration du serveur
Vérification de l'interface réseau
Commande :
ip a
Résultat :
enp0s8
Configuration de l'adresse IP statique
Fichier :
nano /etc/network/interfaces
Redémarrage du réseau
reboot
Vérification
ip a
Résultat :
192.168.50.100/24
________________________________________
Installation et configuration du serveur DHCP
Installation
apt update
apt install isc-dhcp-server 
Définition de l'interface DHCP
nano /etc/default/isc-dhcp-server
Configuration :
INTERFACESv4="enp0s8"
Configuration du service
nano /etc/dhcp/dhcpd.conf

Démarrage du service
systemctl restart isc-dhcp-server
systemctl enable isc-dhcp-server
Vérification
systemctl status isc-dhcp-server
Résultat :
active 
________________________________________
Configuration des postes clients
Configuration DHCP
Fichier :
nano /etc/network/interfaces
Configuration :
auto enp0s8
iface enp0s8 inet dhcp
Renouvellement du bail DHCP
dhclient -r
dhclient
Vérification
ip a
Résultats obtenus :
Machine	Adresse IP
Debianclient1	192.168.50.35
Debianclient2	192.168.50.43
________________________________________
Analyse du protocole DHCP avec Wireshark
Filtre utilisé
dhcp
Procédure
dhclient -r
dhclient
Échanges observés
DHCP Discover
Le client recherche un serveur DHCP.
DHCP Offer
Le serveur propose une adresse IP.
DHCP Request
Le client demande officiellement l'adresse proposée.
DHCP ACK
Le serveur confirme l'attribution.
Capture en annexe
________________________________________
Vérification de la connectivité réseau
Test Serveur ↔ Client
Depuis Debianclient1 :
ping 192.168.50.100
Test Client ↔ Client
Depuis Debianclient1 :
ping 192.168.50.43
Depuis Debianclient2 :
ping 192.168.50.35
Résultat :
Pas de perte
La communication entre toutes les machines est validée.
________________________________________
Installation et configuration du serveur DNS
Installation
apt install bind9 
Déclaration de la zone DNS
nano /etc/bind/named.conf.local
Création de la zone
cp /etc/bind/db.local /etc/bind/db.entreprise
nano /etc/bind/db.entreprise
Redémarrage
systemctl restart bind9
________________________________________
Analyse du protocole DNS avec Wireshark
Configuration des clients
nano /etc/resolv.conf
Configuration :
nameserver 192.168.50.100
search entreprise.local
Test
ping debianclient2.entreprise.local
Filtre Wireshark
dns
Échanges observés
•	DNS Query 
•	DNS Response 
Capture en annexe
________________________________________
Installation et configuration du serveur FTP
Installation depuis Debianserver
apt install vsftpd
Démarrage
systemctl restart vsftpd
systemctl enable vsftpd
Création d'un utilisateur FTP
adduser ftpuser
Test FTP sur debianclient1
Installer filezilla
Apt install filezilla
Puis l’ouvrir
Entrer :
•	utilisateur : ftpuser 
•	mot de passe


Envoi d'un fichier
put test.txt
Téléchargement
get test.txt
Résultat :
Transfert réussi.
________________________________________
Analyse du protocole FTP avec Wireshark
Filtre
ftp
Échanges observés
USER
PASS
STOR
RETR
Capture en annexe
________________________________________
Installation et configuration du serveur HTTP
Installation
apt install apache2
Démarrage
systemctl restart apache2
systemctl enable apache2
Création de la page Web
nano /var/www/html/index.html
Contenu :
<h1>Bienvenue sur ServEntreprise</h1>

Puis l’ouvrir dans le navigateur

Capture en annexe

________________________________________
Analyse du protocole HTTP avec Wireshark
Test
Depuis Debianclient2 :
http://192.168.50.100
ou
http://serventreprise
Filtre Wireshark
http
Échanges observés
GET
HTTP/1.1 200 OK
Capture en annexe
________________________________________
Difficultés rencontrées
Au cours de cette expérimentation, plusieurs problèmes ont été rencontrés :
•	Attribution DHCP impossible suite à une mauvaise interface réseau. 
•	Erreurs de communication dues à une configuration incorrecte du réseau interne VirtualBox. 
•	Problèmes de résolution DNS liés à une erreur dans le fichier de zone. 
•	Message « Destination Host Unreachable » lors des premiers tests de communication. 
Ces difficultés ont été résolues grâce à la vérification des interfaces réseau, des adresses IP et des fichiers de configuration des différents services.
________________________________________
Conclusion
Cette expérience nous a permis de déployer avec succès une infrastructure réseau complète sous Debian. Le serveur ServEntreprise a assuré les services DHCP, DNS, FTP et HTTP pour les deux postes clients du réseau local. Grâce aux analyses menées avec Wireshark, nous avons pu observer le fonctionnement des protocoles réseau étudiés et mieux comprendre les échanges entre les différentes machines. Les tests de communication, de résolution DNS, de transfert FTP et d’accès Web ont confirmé le bon fonctionnement de l’ensemble de l’infrastructure mise en place.
