# Teaching-HEIGVD-SRX-2019-Laboratoire-IDS

**ATTENTION : Commencez par créer un Fork de ce repo et travaillez sur votre fork.**

Clonez le repo sur votre machine. Vous pouvez répondre aux questions en modifiant directement votre clone du README.md ou avec un fichier pdf que vous pourrez uploader sur votre fork.

**Le rendu consiste simplement à répondre à toutes les questions clairement identifiées dans le text avec la mention "Question" et à les accompagner avec des captures. Le rendu doit se faire par une "pull request". Envoyer également le hash du dernier commit et votre username GitHub par email au professeur et à l'assistant**

## Table de matières

[Introduction](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#introduction)

[Echéance](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#echéance)

[Configuration du réseau](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#configuration-du-réseau-sur-virtualbox)

[Installation de Snort](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#installation-de-snort-sur-linux)

[Essayer Snort](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#essayer-snort)

[Utilisation comme IDS](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#utilisation-comme-un-ids)

[Ecriture de règles](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#ecriture-de-règles)

[Travail à effectuer](https://github.com/arubinst/Teaching-HEIGVD-SRX-2019-Laboratoire-IDS#exercises)


## Echéance 

Ce travail devra être rendu le dimanche après la fin de la 2ème séance de laboratoire, soit au plus tard, **le 14 avril 2019, à 23h.**


## Introduction

Dans ce travail de laboratoire, vous allez explorer un système de détection contre les intrusions (IDS) dont l'utilisation es très répandue grâce au fait qu'il est gratuit et open source. Il s'appelle [Snort](https://www.snort.org). Il existe des versions de Snort pour Linux et pour Windows.

### Les systèmes de détection d'intrusion

Un IDS peut "écouter" tout le traffic de la partie du réseau où il est installé. Sur la base d'une liste de règles, il déclenche des actions sur des paquets qui correspondent à la description de la règle.

Un exemple de règle pourrait être, en langage commun : "donner une alerte pour tous les paquets envoyés par le port http à un serveur web dans le réseau, qui contiennent le string 'cmd.exe'". En on peut trouver des règles très similaires dans les règles par défaut de Snort. Elles permettent de détecter, par exemple, si un attaquant essaie d'exécuter un shell de commandes sur un serveur Web tournant sur Windows. On verra plus tard à quoi ressemblent ces règles.

Snort est un IDS très puissant. Il est gratuit pour l'utilisation personnelle et en entreprise, où il est très utilisé aussi pour la simple raison qu'il est l'un des plus efficaces systèmes IDS.

Snort peut être exécuté comme un logiciel indépendant sur une machine ou comme un service qui tourne après chaque démarrage. Si vous voulez qu'il protège votre réseau, fonctionnant comme un IPS, il faudra l'installer "in-line" avec votre connexion Internet. 

Par exemple, pour une petite entreprise avec un accès Internet avec un modem simple et un switch interconnectant une dizaine d'ordinateurs de bureau, il faudra utiliser une nouvelle machine exécutant Snort et placée entre le modem et le switch. 


## Matériel

Vous avez besoin de votre ordinateur avec VirtualBox et une VM Kali Linux. Vous trouverez un fichier OVA pour la dernière version de Kali sur `//eistore1/cours/iict/Laboratoires/SRX/Kali` si vous en avez besoin.


## Configuration du réseau sur VirtualBox

Votre VM fonctionnera comme IDS pour "protéger" votre machine hôte (par exemple, si vous faites tourner VirtualBox sur une machine Windows, Snort sera utilisé pour capturer le trafic de Windows vers l'Internet).

Pour cela, il faudra configurer une réseau de la VM en mode "bridge" et activer l'option "Promiscuous Mode" dans les paramètres avancés de l'interface. Le mode bridge dans l'école ne vous permet pas d'accéder à l'Internet depuis votre VM. Vous pouvez donc rajouter une deuxième interface réseau à votre Kali configurée comme NAT. La connexion Internet est indispensable pour installer Snort mais pas vraiment nécessaire pour les manipulations du travail pratique.

Pour les captures avec Snort, assurez-vous de toujours indiquer la bonne interface dans la ligne de commandes, donc, l'interface configurée en mode promiscuous.

![Topologie du réseau virtualisé](images/Snort_Kali.png)


## Installation de Snort sur Linux

On va installer Snort sur Kali Linux. Si vous avez déjà une VM Kali, vous pouvez l'utiliser. Sinon, vous avez la possibilité de copier celle sur `eistore`.

La manière la plus simple c'est de d'installer Snort en ligne de commandes. Il suffit d'utiliser la commande suivante :

```
sudo apt update && apt install snort
```

Ceci télécharge et installe la version la plus récente de Snort.

Vers la fin de l'installation, on vous demande de fournir l'adresse de votre réseau HOME. Il s'agit du réseau que vous voulez protéger. Cela sert à configurer certaines variables pour Snort. Pour les manipulations de ce laboratoire, vous pouvez donner n'importe quelle adresse comme réponse.


## Essayer Snort

Une fois installé, vous pouvez lancer Snort comme un simple "sniffer". Pourtant, ceci capture tous les paquets, ce qui peut produire des fichiers de capture énormes si vous demandez de les journaliser. Il est beaucoup plus efficace d'utiliser des règles pour définir quel type de trafic est intéressant et laisser Snort ignorer le reste.

Snort se comporte de différentes manières en fonction des options que vous passez en ligne de commande au démarrage. Vous pouvez voir la grande liste d'options avec la commande suivante :

```
snort --help
```

On va commencer par observer tout simplement les entêtes des paquets IP utilisant la commande :

```
snort -v -i eth0
```

**ATTENTION : assurez-vous de bien choisir l'interface qui se trouve en mode bridge/promiscuous. Elle n'est peut-être pas eth0 chez-vous!**

Snort s'execute donc et montre sur l'écran tous les entêtes des paquets IP qui traversent l'interface eth0. Cette interface est connectée à l'interface réseau de votre machine hôte à travers le bridge de VirtualBox.

Pour arrêter Snort, il suffit d'utiliser `CTRL-C`.

## Utilisation comme un IDS

Pour enregistrer seulement les alertes et pas tout le trafic, on execute Snort en mode IDS. Il faudra donc spécifier un fichier contenant des règles. 

Il faut noter que `/etc/snort/snort.config` contient déjà des références aux fichiers de règles disponibles avec l'installation par défaut. Si on veut tester Snort avec des règles simples, on peut créer un fichier de config personnalisé (par exemple `mysnort.conf`) et importer un seul fichier de règles utilisant la directive "include".

Les fichiers de règles sont normalement stockés dans le répertoire `/etc/snort/rules/`, mais en fait un fichier de config et les fichiers de règles peuvent se trouver dans n'importe quel répertoire. 

Par exemple, créez un fichier de config `mysnort.conf` dans le repertoire `/etc/snort` avec le contenu suivant :

```
include /etc/snort/rules/icmp2.rules
```

Ensuite, créez le fichier de règles `icmp2.rules` dans le répertoire `/etc/snort/rules/` et rajoutez dans ce fichier le contenu suivant :

`alert icmp any any -> any any (msg:"ICMP Packet"; sid:4000001; rev:3;)`

On peut maintenant executer la commande :

```
snort -c /etc/snort/mysnort.conf
```

Vous pouvez maintenant faire quelques pings depuis votre hôte et regarder les résultats dans le fichier d'alertes contenu dans le répertoire `/var/log/snort/`. 


## Ecriture de règles

Snort permet l'écriture de règles qui décrivent des tentatives de exploitation de vulnérabilités bien connues. Les règles Snort prennent en charge à la fois l'analyse de protocoles et la recherche et identification de contenu.

Il y a deux principes de base à respecter :

* Une règle doit être entièrement contenue dans une seule ligne
* Les règles sont divisées en deux sections logiques : (1) l'entête et (2) les options.

L'entête de la règle contient l'action de la règle, le protocole, les adresses source et destination, et les ports source et destination.

L'option contient des messages d'alerte et de l'information concernant les parties du paquet dont le contenu doit être analysé. Par exemple:

```
alert tcp any any -> 192.168.1.0/24 111 (content:"|00 01 86 a5|"; msg: "mountd access";)
```

Cette règle décrit une alerte générée quand Snort trouve un paquet avec tous les attributs suivants :

* C'est un paquet TCP
* Emis depuis n'importe quelle adresse et depuis n'importe quel port
* A destination du réseau identifié par l'adresse 192.168.1.0/24 sur le port 111

Le texte jusqu'au premier parenthèse est l'entête de la règle. 

```
alert tcp any any -> 192.168.1.0/24 111
```

Les parties entre parenthèses sont les options de la règle:

```
(content:"|00 01 86 a5|"; msg: "mountd access";)
```

Les options peuvent apparaître une ou plusieurs fois. Par exemple :

```
alert tcp any any -> any 21 (content:"site exec"; content:"%"; msg:"site
exec buffer overflow attempt";)
```

La clé "content" apparait deux fois parce que les deux strings qui doivent être détectés n'apparaissent pas concaténés dans le paquet mais à des endroits différents. Pour que la règle soit déclenchée, il faut que le paquet contienne **les deux strings** "site exec" et "%". 

Les éléments dans les options d'une règle sont traitées comme un AND logique. La liste complète de règles sont traitées comme une succession de OR.

## Informations de base pour le règles

### Actions :

```
alert tcp any any -> any any (msg:"My Name!"; content:"Skon"; sid:1000001; rev:1;)
```

L'entête contient l'information qui décrit le "qui", le "où" et le "quoi" du paquet. Ça décrit aussi ce qui doit arriver quand un paquet correspond à tous les contenus dans la règle.

Le premier champ dans le règle c'est l'action. L'action dit à Snort ce qui doit être fait quand il trouve un paquet qui correspond à la règle. Il y a six actions :

* alert - générer une alerte et écrire le paquet dans le journal
* log - écrire le paquet dans le journal
* pass - ignorer le paquet
* drop - bloquer le paquet et l'ajouter au journal
* reject - bloquer le paquet, l'ajouter au journal et envoyer un `TCP reset` si le protocole est TCP ou un `ICMP port unreachable` si le protocole est UDP
* sdrop - bloquer le paquet sans écriture dans le journal

### Protocoles :

Le champ suivant c'est le protocole. Il y a trois protocoles IP qui peuvent être analysez par Snort : TCP, UDP et ICMP.


### Adresses IP :

La section suivante traite les adresses IP et les numéros de port. Le mot `any` peut être utilisé pour définir "n'import quelle adresse". On peut utiliser l'adresse d'une seule machine ou un block avec la notation CIDR. 

Un opérateur de négation peut être appliqué aux adresses IP. Cet opérateur indique à Snort d'identifier toutes les adresses IP sauf celle indiquée. L'opérateur de négation est le `!`.

Par exemple, la règle du premier exemple peut être modifiée pour alerter pour le trafic dont l'origine est à l'extérieur du réseau :

```
alert tcp !192.168.1.0/24 any -> 192.168.1.0/24 111
(content: "|00 01 86 a5|"; msg: "external mountd access";)
```

### Numéros de Port :

Les ports peuvent être spécifiés de différentes manières, y-compris `any`, une définition numérique unique, une plage de ports ou une négation.

Les plages de ports utilisent l'opérateur `:`, qui peut être utilisé de différentes manières aussi :

```
log udp any any -> 192.168.1.0/24 1:1024
```

Journaliser le traffic UDP venant d'un port compris entre 1 et 1024.

---

```
log tcp any any -> 192.168.1.0/24 :6000
```

Journaliser le traffic TCP venant d'un port plus bas ou égal à 6000.

---

```
log tcp any :1024 -> 192.168.1.0/24 500:
```

Journaliser le traffic TCP venant d'un port privilégié (bien connu) plus grand ou égal à 500 mais jusqu'au port 1024.


### Opérateur de direction

L'opérateur de direction `->`indique l'orientation ou la "direction" du trafique. 

Il y a aussi un opérateur bidirectionnel, indiqué avec le symbole `<>`, utile pour analyser les deux côtés de la conversation. Par exemple un échange telnet :

```
log 192.168.1.0/24 any <> 192.168.1.0/24 23
```

## Alertes et logs Snort

Si Snort détecte un paquet qui correspond à une règle, il envoie un message d'alerte ou il journalise le message. Les alertes peuvent être envoyées au syslog, journalisées dans un fichier text d'alertes ou affichées directement à l'écran.

Le système envoie **les alertes vers le syslog** et il peut en option envoyer **les paquets "offensifs" vers une structure de repertoires**.

Les alertes sont journalisées via syslog dans le fichier `/var/log/snort/alerts`. Toute alerte se trouvant dans ce fichier aura son paquet correspondant dans le même repertoire, mais sous le fichier snort.log.xxxxxxxxxx où xxxxxxxxxx est l'heure Unix du commencement du journal.

Avec la règle suivante :

```
alert tcp any any -> 192.168.1.0/24 111
(content:"|00 01 86 a5|"; msg: "mountd access";)
```

un message d'alerte est envoyé à syslog avec l'information "mountd access". Ce message est enregistré dans /var/log/snort/alerts et le vrai paquet responsable de l'alerte se trouvera dans un fichier dont le nom sera /var/log/snort/snort.log.xxxxxxxxxx.

Les fichiers log sont des fichiers binaires enregistrés en format pcap. Vous pouvez les ouvrir avec Wireshark ou les diriger directement sur la console avec la commande suivante :

```
tcpdump -r /var/log/snort/snort.log.xxxxxxxxxx
```

Vous pouvez aussi utiliser des captures Wireshark ou des fichiers snort.log.xxxxxxxxx comme source d'analyse pour Snort.

## Exercises

**Réaliser des captures d'écran des exercices suivants et les ajouter à vos réponses.**

### Trouver votre nom :

Considérer la règle simple suivante:

alert tcp any any -> any any (msg:"Mon nom!"; content:"Rubinstein"; sid:4000015; rev:1;)

**Question 1: Qu'est-ce qu'elle fait la règle et comment ça fonctionne ?**

---

**Réponse :**  Cette règle génère une alerte et écrit dans le journal si : 

* un paquet avec le protocole TCP est détecté...
* et qu'il provient de n'importe quelle adresse IP depuis n'importe quel port...
* et qu'il va à destination de n'importe quelle adresse IP sur n'importe quel port...
* et qu'il contient "Rubinstein"

Dans ce cas, un message d'alerte est envoyé à syslog avec l'information "Mon nom!". Plus précisément, le message est enregistré dans `/var/log/snort/alerts` et le paquet est stocké dans un fichier nommé `/var/log/snort/snort.log.xxxxxxxxxx`.

L'ID unique de cette règle est 4000015 et le numéro de révision est 1.

---

Utiliser un éditeur et créer un fichier `myrules.rules` sur votre répertoire home. Rajouter une règle comme celle montrée avant mais avec votre nom ou un mot clé de votre préférence. Lancer snort avec la commande suivante :

```
sudo snort -c myrules.rules -i eth0
```

**Question 2: Que voyez-vous quand le logiciel est lancé ? Qu'est-ce que ça vaut dire ?**

---

**Réponse :**  Une fois lancé, Snort affiche d'abord des informations d'initialisations, puis il commence à analyser les paquets. Aucune option d'affichage n'est renseignée, donc aucune information relative au trafic n'apparaît sur la console. 

Cependant, vu qu'aucun pré-processeur n'est configuré dans notre fichier, Snort affiche en boucle le message _WARNING: No preprocessors configured for policy 0._ à intervalle très court. Ce message apparaît car notre fichier de configuration ne contient aucun pré-processeur, qui sont des "plug-ins" de Snort qui permettent de suivre des connexions, réassembler les paquets, décoder certains types de protocoles, etc. 

Ces pré-processeurs sont configurés dans le fichier de configuration `/etc/snort/snort.conf` fourni par défaut.

---

Aller à un site web contenant votre nom ou votre mot clé que vous avez choisi dans son texte (il faudra chercher un peu pour trouver un site en http...). Ensuite, arrêter Snort avec `CTRL-C`.

**Question 3: Que voyez-vous ?**

---

**Réponse :**  Dans la console, toujours aucune information sur le trafic, mais plein de _warnings preprocessors_. Une fois l'analyse terminée avec `CTRL-C`, un récapitulatif s'affiche avec, entre autres :

* la durée de la capture
* le nombre de paquets traités, analysés
* les protocoles rencontrés
* les actions entreprises
* etc.

---

Aller au répertoire /var/log/snort. Ouvrir le fichier `alert`. Vérifier qu'il y ait des alertes pour votre nom.

**Question 4: A quoi ressemble l'alerte ? Qu'est-ce que chaque champ veut dire ?**

---

**Réponse :**  Une alerte reçue a la forme suivante :

```
[**] [1:4555555:1] Petit chat mignon! [**]
[Priority: 0] 
04/02-16:26:45.736104 10.192.106.82:51645 -> 23.8.1.34:80
TCP TTL:128 TOS:0x0 ID:64396 IpLen:20 DgmLen:481 DF
***AP*** Seq: 0x92176D80  Ack: 0x6D065DD6  Win: 0x201  TcpLen: 20
```

Le premier champs contient l'ID de notre règle ainsi que sa révision et son message.

Le second champs renseigne sur la priorité, ici `0` signifie la plus basse. Snort se réfère au fichier `classification.config` pour connaître la priorité d'un évènement.

Le troisième champs indique entre autres la date de l'alerte ainsi que les adresses IP concernées.

Les quatrième et cinquième champs donnent diverses informations sur le paquet en lui-même.

---

### Détecter une visite à Wikipédia

Ecrire une règle qui journalise (sans alerter) un message à chaque fois que Wikipédia est visité **DEPUIS VOTRE** station. **Ne pas utiliser une règle qui détecte un string ou du contenu**.

**Question 5: Quelle est votre règle ? Où le message a-t'il été journalisé ? Qu'est-ce qui a été journalisé ?**

---

**Réponse :**  Notre règle est la suivante :

```
log tcp 10.192.106.82 any -> 91.198.174.192 [80,443] (sid:4555556; rev:1;) 
```

Pendant la capture, avec le paramètre `-v`, on peut constater des messages tels que celui-ci sur la console :

```
04/04-17:12:16.849014 10.192.106.82:59234 -> 91.198.174.192:443
```

Une fois la capture terminée, on se rend à l'emplacement `/var/log/snort` et on peut constater un nouveau fichier de log nommé `snort.log.1554390576` que l'on peut ouvrir avec Wireshark. Il contient l'échange complet des paquets entre l'hôte et Wikipédia.

---

### Détecter un ping d'un autre système

Ecrire une règle qui alerte à chaque fois que votre système reçoit un ping depuis une autre machine. Assurez-vous que **ça n'alerte pas** quand c'est vous qui envoyez le ping vers un autre système !

**Question 6: Quelle est votre règle ? Comment avez-vous fait pour que ça identifie seulement les pings entrants ? Où le message a-t'il été journalisé ? Qu'est-ce qui a été journalisé ?**

---

**Réponse :**  Notre règle est la suivante :

```
alert icmp any any -> 10.192.106.82 any (itype:8; msg:"I have been ping'd!"; sid:4555557; rev:1;)
```

Pour identifier seulement les pings entrants, il suffit de spécifier que l'on ne veut loguer que les `ECHO REQUEST`  sur l'IP hôte avec le paramètre `itype:8` et de bien mettre l'hôte en destinataire (->).

Le message est journalisé dans le fichier `/var/log/snort/alert` et on voit les informations suivantes :

```
[**] [1:4555557:1] I have been ping'd! [**]
[Priority: 0] 
04/04-17:36:12.168580 10.192.95.109 -> 10.192.106.82
ICMP TTL:127 TOS:0x0 ID:15835 IpLen:20 DgmLen:60
Type:8  Code:0  ID:1   Seq:8  ECHO
```

Sans le paramètre `itype:8`, les `ECHO REPLY` sont aussi journalisées.

---

### Détecter les ping dans les deux sens

Modifier votre règle pour que les pings soient détectés dans les deux sens.

**Question 7: Qu'est-ce que vous avez modifié pour que la règle détecte maintenant le trafic dans les deux sens ?**

---

**Réponse :**  Pour identifier les `ECHO REQUEST` dans les deux sens, on modifie la règle comme suit :

```
alert icmp any any <> 10.192.106.82 any (itype:8; msg:"Someone is playing ping without pong!";sid:4555557; rev:1;)
```

L'opérateur de direction `<>` permet d'activer la règle dans les deux sens, c'est à-dire que la source et la destination peuvent être intervertis. Cela fonctionne à présent dans les deux sens :

```
[**] [1:4555557:1] Someone is playing ping without pong! [**]
[Priority: 0] 
04/04-17:44:12.555070 10.192.95.109 -> 10.192.106.82
ICMP TTL:127 TOS:0x0 ID:16410 IpLen:20 DgmLen:60
Type:8  Code:0  ID:1   Seq:13  ECHO

[**] [1:4555557:1] Someone is playing ping without pong! [**]
[Priority: 0] 
04/04-17:44:13.559548 10.192.106.82 -> 10.192.95.109
ICMP TTL:128 TOS:0x0 ID:7478 IpLen:20 DgmLen:60
Type:8  Code:0  ID:1   Seq:52  ECHO
```



---

### Détecter une tentative de login SSH

Essayer d'écrire une règle qui alerte qu'une tentative de session SSH a été faite depuis la machine d'un voisin. Si vous avez besoin de plus d'informations sur ce qui décrit cette tentative (adresses, ports, protocoles), servez-vous de Wireshark pour analyser les échanges lors de la requête de connexion depuis votre voisin.

**Question 8: Quelle est votre règle ? Montrer la règle et expliquer comment elle fonctionne. Montre le message d'alerte enregistré dans le fichier d'alertes.**

---

**Réponse :**  Notre règle est la suivante :

```
alert tcp 10.192.0.0/16 any -> 10.192.106.82 22 (msg:"ssh Login Attempt"; sid:4555558; rev:1;)
```

Elle fonctionne comme suit : si une adresse IP du range 10.192.0.1 à 10.192.255.254 (range de l'école ?) tente une connexion SSH sur l'hôte (donc sur son port 22), alors Snort le détecte et log une alerte.

```
[**] [1:4555558:1] ssh Login Attempt [**]
[Priority: 0] 
04/04-18:00:38.918790 10.192.95.109:60711 -> 10.192.106.82:22
TCP TTL:127 TOS:0x0 ID:17563 IpLen:20 DgmLen:52 DF
******S* Seq: 0x85AADBFD  Ack: 0x0  Win: 0xFAF0  TcpLen: 32
TCP Options (6) => MSS: 1460 NOP WS: 8 NOP NOP SackOK 
```



---

### Analyse de logs

Lancer Wireshark et faire une capture du trafic sur l'interface connectée au bridge. Générez du trafic avec votre machine hôte qui corresponde à l'une des règles que vous avez ajouté à votre fichier de configuration personnel. Arrêtez la capture et enregistrez-la dans un fichier.

**Question 9: Quelle est l'option de Snort qui permet d'analyser un fichier pcap ou un fichier log ?**

---

**Réponse :**  La commande suivante permet d'analyser un fichier de log pcap :

```
snort -r snort.pcap
```

---

Utiliser l'option correcte de Snort pour analyser le fichier de capture Wireshark.

**Question 10: Quelle est le comportement de Snort avec un fichier de capture ? Y-a-t'il une différence par rapport à l'analyse en temps réel ? Est-ce que des alertes sont aussi enregistrées dans le fichier d'alertes?**

---

**Réponse :**  L'ouverture avec Snort d'un fichier capturé depuis Wireshark affiche le même "récapitulatif" que lorsqu'il s'agit d'une capture Snort. Cependant, la capture Snort affiche davantage d'informations puisqu'on  peut y voir les statistiques et verdicts à la fin du scan (nombre d'alertes, nombre de logs, nombre de paquets acceptés, etc.).

Contrairement à l'analyse en temps réel, cette capture n'a généré aucune alerte dans les logs vu que Snort ne tournait pas à ce moment-là.

---

<sub>This guide draws heavily on http://cs.mvnu.edu/twiki/bin/view/Main/CisLab82014</sub>