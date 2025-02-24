[Livrables](#livrables)

[Échéance](#échéance)

[Travail à réaliser](#travail-à-réaliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WEP

__A faire en équipes de deux personnes__

### Pour cette partie pratique, vous devez être capable de :

* Déchiffrer manuellement des trames WEP utilisant Python et Scapy
* Chiffrer manuellement des trames WEP utilisant Python et Scapy
* Forger des fragments protégés avec WEP afin d’obtenir une keystream de longueur plus grande que 8 octets


Vous allez devoir faire des recherches sur internet pour apprendre à utiliser Scapy. __Il est fortement conseillé d'employer une distribution Kali__ (on ne pourra pas assurer le support avec d'autres distributions). 


## Travail à réaliser

### 1. Déchiffrement manuel de WEP

Dans cette partie, vous allez récupérer le script Python [manual-decryption.py](files/manual-decryption.py). Il vous faudra également le fichier de capture [arp.cap](files/arp.cap) contenant un message arp chiffré avec WEP et la librairie [rc4.py](files/rc4.py) pour générer les keystreams indispensables pour chiffrer/déchiffrer WEP. Tous les fichiers doivent être copiés dans le même répertoire local sur vos machines.

- Ouvrir le fichier de capture [arp.cap](files/arp.cap) avec Wireshark
   
- Utiliser Wireshark pour déchiffrer la capture. Pour cela, il faut configurer dans Wireshark la clé de chiffrement/déchiffrement WEP (Dans Wireshark : Preferences&rarr;Protocols&rarr;IEEE 802.11&rarr;Decryption Keys). Il faut également activer le déchiffrement dans la fenêtre IEEE 802.11 (« Enable decryption »). Vous trouverez la clé dans le script Python [manual-decryption.py](files/manual-decryption.py).

Résultat : 
   
![Déchiffrement avec Wireshark](figures/1_wireshark_decrypt.png)

- Exécuter le script avec `python manual-decryption.py`

Résultat du script :

```
$ python manual-decryption.py 
Text: aaaa03000000080600010800060400019027e4ea61f2c0a80164000000000000c0a801c8
icv:  cc88cbb2
icv(num): (3431517106,)
```

   
- Comparer la sortie du script avec la capture text déchiffrée par Wireshark

On peut constater que les sorties sont similaires.
   
- Analyser le fonctionnement du script

Analyse du script manual-decryption.py :

Le script charge le fichier arp.cap puis initialise la seed qui est composée de
l'IV concaténée à la clé WEP, qui est 'AA:AA:AA:AA:AA' (sans les ':'). Le script
récupère ensuite l'ICV qui est envoyé avec la trame mais chiffré. Le message
ainsi que l'ICV sont ensuite passé dans RC4 être déchiffrés grâce à la seed. Le
tout est ensuite affiché.

### 2. Chiffrement manuel de WEP

Utilisant le script [manual-decryption.py](files/manual-decryption.py) comme guide, créer un nouveau script `manual-encryption.py` capable de chiffrer un message, l’enregistrer dans un fichier pcap et l’envoyer.
Vous devrez donc créer votre message, calculer le contrôle d’intégrité (ICV), et les chiffrer (voir slides du cours pour les détails).

Résultats : 

Script `files/manual-encryption.py`

Fichier généré par le script: `files/encrypted.pcap`

On peut constater que Wireshark est en mesure de déchiffrer la trame et aucune
erreur n'est détectée :

![Déchiffrement avec Wireshark](figures/2_wireshark_decrypt.png)

### Quelques éléments à considérer :

- Vous pouvez utiliser la même trame fournie comme « template » pour votre trame forgée (conseillé). Il faudra mettre à jour le champ de données qui transporte le message (`wepdata`) et le contrôle d’intégrite (`icv`).
- Le champ `wepdata` peut accepter des données en format text mais il est fortement conseillé de passer des bytes afin d'éviter les soucis de conversions.
- Le champ `icv` accepte des données en format « long ».
- Vous pouvez vous guider à partir du script fourni pour les différentes conversions de formats qui pourraient être nécessaires.
- Vous pouvez exporter votre nouvelle trame en format pcap utilisant Scapy et ensuite, l’importer dans Wireshark. Si Wireshark est capable de déchiffrer votre trame forgée, elle est correcte !


### 3. Fragmentation

Dans cette partie, vous allez enrichir votre script développé dans la partie précédente pour chiffrer 3 fragments.

### Quelques éléments à considérer :

- Chaque fragment est numéroté. La première trame d’une suite de fragments a toujours le numéro de fragment à 0. Une trame entière (sans fragmentation) comporte aussi le numéro de fragment égal à 0
- Pour incrémenter le compteur de fragments, vous pouvez utiliser le champ « SC » de la trame. Par exemple : `trame.SC += 1`
- Tous les fragments sauf le dernier ont le bit `more fragments` à 1, pour indiquer qu’un nouveau fragment va être reçu
- Le champ qui contient le bit « more fragments » est disponible en Scapy dans le champ `FCfield`. Il faudra donc manipuler ce champ pour vos fragments. Ce même champ est visible dans Wireshark dans IEEE 802.11 Data &rarr; Frame Control Field &rarr; Flags
- Pour vérifier que cette partie fonctionne, vous pouvez importer vos fragments dans Wireshark, qui doit être capable de les recomposer
- Pour un test encore plus intéressant (optionnel), vous pouvez utiliser un AP (disponible sur demande) et envoyer vos fragments. Pour que l’AP accepte vous données injectées, il faudra faire une « fake authentication » que vous pouvez faire avec `aireplay-ng`
- Si l’AP accepte vos fragments, il les recomposera et les retransmettra en une seule trame non-fragmentée !


Résultats:

Script: `files/manual-fragmentation.py`

Fichier créé par le script: `files/fragmented.pcap`

On peut constater que Wireshark est en mesure de déchiffrer et défragmenter
correctement la trame : 

![Déchiffrement et défragmentation avec Wireshark](figures/3_wireshark_decrypt_defrag.png)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	[X] Script de chiffrement WEP **abondamment commenté/documenté**
  - [X] Fichier pcap généré par votre script contenant la trame chiffrée
  - [X] Capture d’écran de votre trame importée et déchiffré par Wireshark
-	[X] Script de fragmentation **abondamment commenté/documenté**
  - [X] Fichier pcap généré par votre script contenant les fragments
  - [X] Capture d’écran de vos trames importées et déchiffrés par Wireshark 

-	Envoyer le hash du commit et votre username GitHub par email au professeur et à l'assistant


## Échéance

Le 20 avril 2023 à 13h15
