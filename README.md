# Synthèse
Cet article à pour but de parler des utilisations malveillantes des infrastructures CDN.

<center> <H1> CDN </H1> </center>

Un Content distribution network où dit CDN est un groupe de serveurs qui se partagent des données en cache et permet le transfert rapide des ressources nécessaires au chargement d'un contenu sur Internet, *Par exemple: des pages HTML, des fichiers, des images et des vidéos.* 
- Il permet d'améliorer grandement la qualité d'experience de l'utilisateur final, c'est pour cette raison que souvent il est utilisé dans des déploiements à grande échelle pour permettre de délivrer un contenu rapidement sans tenir compte de la localisation de l'utilisateur final.
- Il permet de réduire le coût de la bande passante pour les hébergeurs de contenu web.
- Un CDN peut aider à prémunir une ressource réseau d'une surcharge face aux attaques DDOS ce qui peut provoquer une interruption de service. Dans ce cas la un CDN va répartir la charge des requêtes reçues entre les différents serveurs appartenant a cette meme infrastructure.

Un serveur qui fait office de CDN à deux modes distincts : push et pull
En mode push la personne qui à lié son site web au CDN va upload ses ressources en avance.
En mode pull le serveur CDN va marcher comme un reverse proxy en redistribuant les données via le cache

<center> <H1> Différentes attaques et cas pratiques </H1> </center>

## Avantages pour les Malware et C&C 

- Fournis une bande passante importante pour la distribution de malware
	- **Exemple avec la distribution d'un fichier malveillant sur le CDN de discord: https://tehtris.com/fr/blog/ukraine-quel-est-donc-ce-malware-destructeur-deguise-en-ransomware**
- Garantis que les tentatives de mise hors service ne sont pas toujours effectives du premier coup et sur des CDN fréquemment utilisés il est impraticable de bloquer le domaine du CDN.
- Il peut être difficile de trouver des IOC dans certains cas car il peut y avoir un grand nombre d'accès fait par des services légitimes sur le meme CDN.
- Peux chiffrer le traffic avec le le vrai certificat du CDN.

## MITM (man-in-the-middle)

Un malware peut être injecté lors d'un réponse HTTP venant d'un CDN qui lui meme est compromis.

![Pasted image 20230506014046](https://github.com/routeur/Documentation_CDN/assets/49996859/ef44729e-cff8-4d3b-b871-c3b613adba56)

Dans notre cas le nœud A correspond à un CDN amont qui redistribue le contenu mais qui le possède pas contrairement à l'infrastructure compromise qui est elle un CDN aval et qui possède le contenu.

**Possibilités pour un CDN compromis :**
- Injecter du code coté client dans des réponses proxy-fiées pour récupérer des mots de passes ou traquer des machines clientes.
- Modifier le contenu d'une page web pour y inclure des publicités.

## Types d'abus
Voici les différents types d'abus qui peuvent être commis lors de l'utilisation d'un CDN.

**HTTP-header manipulation :**
- Permet le changement du champ **Host** du http-header pour changer la destination du packet lors du transit dans l'infrastructure du CDN.
- Permet de changer le champ **X-Forwarded-For** est un entête http permettant d'indiquer l'adresse IP du client d'origine et peut servir à faire croire qu'une requête proviens d'une adresse IP légitime. 
- Reset d'un header HTTP , qui permet à un attaquant de supprimer le **Referer** qui indique la source de la requête et éviter d’être traqué.

**Origin Port abuse :**
- Certains CDN supportent la configuration de ports dans des range d'IP assez large pour permettre  à un attaquant de faire des requêtes sur différents ports ,la ou ça deviens intéressant c'est que le CDN utilisé va répondre par **Filtered, Closed or Open**, une minorité de CDN provider ne le fait pas et retourne des réponses non distinctives sur le status des ports.
- TCP concurrent connection exhaustion attack permet à un attaquant d'abuser du taux de charge maximal accepté par une machine pour faire du DoS , dans ce cas la je parle d'une surcharge de requêtes DNS vers une machine qui utilise un service autre que HTTP sur un site d'origine.

### Proxy
**Forwarding abuse :**
- Pointe l'origine de la requête vers un autre site (cette technique est utilisée pour échapper à la censure)

**Segment TCP abuse :**
**Explication :** Les deux segments TCP (front-end et back-end) crées par le CDN peuvent avoir un état TCP incohérent, par exemple le back-end peut être en transmission tandis que le front end peut avoir fermé sa connexion .
- Attaque de déni de service par épuisement de la bande passante contre les serveurs de site Web alimentés par un CDN.
Elle consiste à submerger de requêtes le front end d'un site qui à taux d'acceptation de la charge qui est réduite ce qui résulte à un DOS.

### Geo-IP
**IP Abondance abuse :**
Il est possible d'avoir accès à du contenu en indiquant une source complètement différente.

![Pasted image 20230507004410](https://github.com/routeur/Documentation_CDN/assets/49996859/3016bb68-cc11-40cd-8390-bcdb2c41131f)

Schéma assez généraliste sur une requête faite d'un utilisateur pour accéder à une ressource.
Il est possible de bypass les restrictions par IP en multipliant les requêtes en passant par un CDN. 

## Forwarding-loop Attack

En général le CDN fait office de relais pour atteindre une destination, lorsqu'une requête est faite le CDN regarde dans le "host header" de la requête HTTP qui va lui permettre de "forward" la requête vers une destination définie en amont qui sera le serveur le plus proche de la destination. Pour redistribuer du contenu, un attaquant va identifier le nœud le plus proche du site cible et faire une requête HTTP sur ce nœud cela va initier une boucle au niveau de ce même nœud et va se redistribuer la requête HTTP.

**Il y a plusieurs types de boucles :**
- Self loop : Le nœud boucle la requête sur lui même
- Intra-CDN loop : Fait une boucle sur des nœuds d'un même CDN
- Inter-CDN loop : Fait un nœud sur plusieurs CDN différents
- Dam-flooding : l'attaquant (client) va vouloir envoyer une requête HTTP sur son site , la requête seras "Forward" dans les nœuds transitaires pour retourner au point de départ et interroger le serveur de destination qui lui va transmettre une réponse Stream qui va provoquer un effet de boucle amplifié sur tous les nœuds transitaires du CDN avant de répondre a l'attaquant (client). Pour augmenter le facteur d'amplification il est possible d'intégrer à la requête envoyé au serveur de destination une gzip bomb avec le header "Accept-Encoding:identity" afin que le nœud 0 du CDN le décompresse .

<center> <H1> Conclusion </H1> </center> 

<center> <H1> Terminologies </H1> </center> 

**IOC :**
- Indicator of compromise 

**Authoritative CDN :**
- Un CDN qui à un lien direct avec un CSP pour la distribution et l'envoi d'un contenu du CSP par l'Authoritative CDN ou par liaison CDN descendante de l'authoritative CDN.(https://www.rfc-editor.org/rfc/rfc6707)

**CDN interconnection (CDNI) :** 
- Liaison inter-CDN permet au service de distribution de contenu à partir de X(Downstream CDN) au nom d'un CDN Y(Upstream CDN).
(https://www.rfc-editor.org/rfc/rfc6707)

**Domaine CDN :** 
- Le nom de domaine dans l'URL qui identifie le CDN. Par exemple, pour le site Web de l'Élysée, qui utilise le CDN de Level 3, c'est actuellement cdn.cdn-tech.com.c.footprint.net. (https://www.bortzmeyer.org/7336.html)

**Redirection Itérative :**
- Lorsque le CDN amont renvoie simplement au CDN aval, sans essayer de traiter la requête lui-même,

**Redirection Récursive :**
- Lorsque le CDN amont fait le travail de redirection lui-même, sans que le client qui accédait au contenu en soit même conscient.

<center> <H1> References </H1> </center> 
- https://community.akamai.com/customers/s/question/0D54R000074TaJLSA0/should-origin-certificate-be-pinned-at-akamai-for-better-security?language=en_US
- https://www.internetsociety.org/wp-content/uploads/2021/01/forwarding-loop-attacks-content-delivery-networks.pdf
- https://www.securityweek.com/new-attack-abuses-cdns-spread-malware/
- https://youtu.be/txKdD9FlFBs
- https://www.cyberark.com/resources/threat-research-blog/implementing-malware-command-and-control-using-major-cdns-and-high-traffic-domains
- https://www.bamsoftware.com/papers/fronting/
- https://www.bortzmeyer.org/7336.html
- https://www.amitlevy.com/papers/stickler-w2sp15.pdf
- https://www.jianjunchen.com/p/cdn-origin-validation.SDRS18.pdf
- Internet Web Replication and Caching Taxonomy (https://www.rfc-editor.org/rfc/rfc3040)
- Content Distribution Network Interconnection (CDNI) Problem Statement(https://www.rfc-editor.org/rfc/rfc6707)
- https://www.akamai.com/fr/our-thinking/ddos
- https://www.belugacdn.com/content-delivery-network-security
