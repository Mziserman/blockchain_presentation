
---
# Registre

---
# Registre
## Protocol
- On communique toute nouvelle dette a tous les membres de Lifen
- Tous les membres gardent un registre avec une liste de toutes les dettes du mois
- Tous les premiers du mois on règle les dettes

---
# Registre
## Example


```
┌────────┐                                  ┌────────┐
│ MARTIN │ ───────────────────────────────► │ ARTHUR │
└────────┘                                  └────────┘
              ┌───────────────────────┐
              │{                      │
              │  creditor: "Aymeric", │
              │  debitor: "Martin",   │
              │  amount: 500          │
              │}                      │
              └───────────────────────┘

┌────────┐                                  ┌───────────┐
│ MARTIN │ ───────────────────────────────► │ ALEXANDRE │
└────────┘                                  └───────────┘
              ┌───────────────────────┐
              │{                      │
              │  creditor: "Aymeric", │
              │  debitor: "Martin",   │
              │  amount: 500          │
              │}                      │
              └───────────────────────┘

┌────────┐                                  ┌────────┐
│ MARTIN │ ───────────────────────────────► │ VICTOR │
└────────┘                                  └────────┘
              ┌───────────────────────┐
              │{                      │
              │  creditor: "Aymeric", │
              │  debitor: "Martin",   │
              │  amount: 500          │
              │}                      │
              └───────────────────────┘

┌────────┐                                  ┌─────────┐
│ MARTIN │ ───────────────────────────────► │ AYMERIC │
└────────┘                                  └─────────┘
              ┌───────────────────────┐
              │{                      │
              │  creditor: "Aymeric", │
              │  debitor: "Martin",   │
              │  amount: 500          │
              │}                      │
              └───────────────────────┘

```
---
# Registre
## Avantages
- Simple et fonctionnel temps que tout le monde est honnête et qu'on a pas trop de transactions
---
# Registre
## Désavantages
- Basé sur la confiance :
  - On ne vérifie pas que l'émetteur a l'autorisation de créer la dette  
  - On se fait assez confiance pour que tout le monde rembourse a la fin du mois
- Chaque transactions doivent être communiquées à tout le monde
---
# Registre
## Attaques
- Un utilisateur peut envoyer des dettes des autres utilisateurs vers lui
```
┌────────┐                                  ┌─────────┐
│ MARTIN │ ───────────────────────────────► │ AYMERIC │
└────────┘                                  └─────────┘
              ┌───────────────────────┐
              │{                      │
              │  creditor: "Martin",  │
              │  debitor: "Arthur",   │
              │  amount: 5_000_00     │
              │}                      │
              └───────────────────────┘

```
- Un utilisateur peut engranger des milliers d'euros de dettes et arrêter d'utiliser le registre sans rembourser
- Un utilisateur peut engranger une dette auprés d'un utilisateur et n'envoyer la requete qu'a cet utilisateur sans prevenir les autres
---
# Registre
## Solutions
### Verification de la provenance de la requête
#### Public/Private keys
```
PK: Public key
SK: Secret key
```
- Un couple de clés publique/privée permet de signer et de vérifier les signature : 
```bash
sign(data, SK) => signature
verify(data, signature, PK) => bool
# Si la signature a été créée avec la data et la clé privée liée a la clé publique utilisée pour vérifier, true, sinon false
```
- On ne peut signer qu'avec une clé privée
- Il est infaisable (pas impossible) de falsifier une signature sans connaitre la clé privée
---
# Registre
## Solutions
### Verification de la provenance de la requête
#### Public/Private keys
##### Implementation
- Chaque utilisateur du registre crée un couple de clé publique/clé privée et rend sa clé publique accessible aux autres utilisateurs
- On rajoute deux informations aux requêtes : 
  - La signature du débiteur
  - Un id unique (qui rend la donnée des transactions uniques permettant d'éviter a un autre membre de dupliquer la transaction et d'avoir une signature valide)

---
# Registre
## Solutions
### Verification de la provenance de la requête
#### Public/Private keys
##### Example

```
┌────────┐                                  ┌─────────┐
│ ARTHUR │ ───────────────────────────────► │ AYMERIC │
└────────┘                                  └─────────┘
              ┌───────────────────────┐
              │{                      │
              │  id: 1,               │
              │  creditor: "Martin",  │
              │  debitor: "Arthur",   │
              │  amount: 5_000_00,    │
              │  signature: 10110101  │ <- si cette signature n'est pas verifiable avec la clé publique d'Arthur, la transaction n'est pas valide
              │}                      │
              └───────────────────────┘
```
---
# Registre
## Désavantages
- Basé sur la confiance :
  - ~~On ne vérifie pas que l'émetteur a l'autorisation de créer la dette~~
  - On se fait assez confiance pour que tout le monde rembourse a la fin du mois
- Chaque transactions doivent être communiquées à tout le monde

---
# Registre
## Solutions
### Empêcher les utilisateur de manquer le remboursement
#### Lifen Coin
- Un meilleur moyen de s'assurer que les débiteur remboursent leur créditeur est d'avoir une monnaie interne au registre. 
- Au lieu d'enregistrer les dettes, le registres enregistre les transactions en monnaie interne.
- Le Lifen Coin est échangeable à un taux de 1LC pour 0.01€ dans les deux sens
- La première transaction de chaque utilisateurs du registre est le buy-in
- Un utilisateur ne peut créer de transaction si il n'a pas les fonds (verifiable grâce aux transactions précédentes dans le registre)
  - somme des buy ins + somme des montants reçues - somme des montants envoyés = fonds disponibles
```
┌───────────┐                                  ┌─────────┐
│ ALEXANDRE │ ───────────────────────────────► │ AYMERIC │
└───────────┘                                  └─────────┘
                 ┌───────────────────────┐
                 │{                      │
                 │  id: 1,               │
                 │  from: 'bank',        │ <- from bank = buy in
                 │  to: 'Arthur',        │
                 │  amount: 50_00,       │ <- Arthur a acheté 50€ de Lifen Coin
                 │  signature: 10110101  │ 
                 │}                      │
                 └───────────────────────┘
```
---
# Registre
## Solutions
### Empêcher les utilisateur de manquer le remboursement
#### Lifen Coin
- Le fonctionnement est le même qu'avant sauf qu'au lieu de régler les dettes le premier du mois, les utilisateur du registre règlent leur dettes en echangeant des Lifen Coins.
- La valeur du Lifen coin est directement liée a l'euro puisque un Lifen coin ne peut être créé qu'en l'échangeant contre 1 centime d'euro.
- Une transaction n'est valide que si :
  - L'émetteur a les fonds
  - La transaction est signée par l'émetteur

```
┌────────┐                                  ┌─────────┐
│ ARTHUR │ ───────────────────────────────► │ MARTIN  │
└────────┘                                  └─────────┘
              ┌───────────────────────┐
              │{                      │
              │  id: 1,               │
              │  from: 'Arthur',      │
              │  to: 'Aymeric',       │
              │  amount: 50_00,       │
              │  signature: 11011001  │
              │}                      │
              └───────────────────────┘
```
---
# Registre
## Désavantages
- ~~Basé sur la confiance :~~
  - ~~On ne vérifie pas que l'émetteur a l'autorisation de créer la dette~~
  - ~~On se fait assez confiance pour que tout le monde rembourse a la fin du mois~~
- Chaque transactions doivent être communiquées à tout le monde
  - pas scalable 
  - les transactions dépendent de l'ordre et les utilisateurs n'ont pas de moyen facile de savoir que ils ont le même ordre que les autres
  - toujours basé sur la confiance, mais plus qu'un seul point of failure (la banque)
---
# Registre
## Solutions
### Cryptographic hash functions
- Une hash function prend en input de la donnée de n'importe quelle taille et sort de la donnée a taille fix qui a l'air aléatoir
- La donnée n'est pas vraiment aléatoire, le même input produira toujours le même output
- Un changement minuscule dans la donnée en entrée produira un gros changement dans la donnée en sortie
- Il est infaisable de retrouver la donnée en entrée grâce a la donnée en sortie
---
# Registre
## Solutions
### Délégation de la verification aux mineurs
#### Blocks
- Au lieu d'avoir une liste de transactions dans le registre, on a une liste de blocks
- Chaque blocks est composé d'une liste de transaction couplé a un nonce
  - un nonce est un nombre qui est ajouté au block et qui fait que le hash du block commence par un certain nombre de 0s
- Le seul moyen de trouver le nonce est de brute-force (essayer avec 1, puis 2, puis 3, puis 4, etc.)
- La première personne qui trouve un nonce générant un hash commençant par le bon nombre de 0 partage son block au reste du registre.
- C'était difficile pour lui de trouver le nonce, mais pour tous les autres utilisateurs, ils ont juste à verifier que le hash du block commence par le bon nombre de 0 
- L'interêt de fournir ce travail est que la première transaction du bloque est une récompense de 100 LC a l'utilisateur qui a créé le bloque

```
┌────────────────────────────┐
│NONCE                       │
├────────────────────────────┤
│TX 1 - COINBASE             │
│TX 2                        │
│TX 3                        │
│...                         │
│TX 100                      │
└─────────────┬──────────────┘
              │ HASH               
┌─────────────▼──────────────┐
│0000000000000011100110101001│
└────────────────────────────┘
```
---
# Registre
## Solutions
### Délégation de la verification aux mineurs
#### BlockChain
- On rajoute le hash du block d'avant dans le block d'après
```
┌────────────────────────────┐       ┌────────────────────────────┐       ┌────────────────────────────┐
│                            │   ┌──►│0000000000000010001110101001│   ┌──►│0000000000000011100110101001│
├────────────────────────────┤   │   ├────────────────────────────┤   │   ├────────────────────────────┤
│NONCE                       │   │   │NONCE                       │   │   │NONCE                       │
├────────────────────────────┤   │   ├────────────────────────────┤   │   ├────────────────────────────┤
│TX 1 - COINBASE             │   │   │TX 1 - COINBASE             │   │   │TX 1 - COINBASE             │
│TX 2                        │   │   │TX 2                        │   │   │TX 2                        │
│TX 3                        │   │   │TX 3                        │   │   │TX 3                        │
│...                         │   │   │...                         │   │   │...                         │
│TX 100                      │   │   │TX 100                      │   │   │TX 100                      │
└─────────────┬──────────────┘   │   └─────────────┬──────────────┘   │   └─────────────┬──────────────┘
              │ HASH             │                 │ HASH             │                 │ HASH
┌─────────────▼──────────────┐   │   ┌─────────────▼──────────────┐   │   ┌─────────────▼──────────────┐
│0000000000000010001110101001├───┘   │0000000000000011100110101001├───┘   │0000000000000011100110101100│
└────────────────────────────┘       └────────────────────────────┘       └────────────────────────────┘
```
- un block est valide si :
  - les transactions qui le composent sont :
    - signées par le payeur
    - le payeur a les fonds
  - il a le hash de son parent
  - le hash du block commence par dix 0
  - (il est sur la chaine la plus longue)
  
---
# Registre
## Solutions
### Délégation de la verification aux mineurs
#### BlockChain
##### Bonus de sécurité
```
┌────────────────────────────┐       ┌────────────────────────────┐       ┌────────────────────────────┐
│                            │   ┌──►│0000000000000010001110101001│   ┌──►│0000000000000011100110101001│
├────────────────────────────┤   │   ├────────────────────────────┤   │   ├────────────────────────────┤
│NONCE                       │   │   │NONCE                       │   │   │NONCE                       │
├────────────────────────────┤   │   ├────────────────────────────┤   │   ├────────────────────────────┤
│TX 1 - COINBASE             │   │   │TX 1 - COINBASE             │   │   │TX 1 - COINBASE             │
│TX 2                        │   │   │TX 2                        │   │   │TX 2                        │
│TX 3                        │   │   │TX 3                        │   │   │TX 3                        │
│...                         │   │   │...                         │   │   │...                         │
│TX 100                      │   │   │TX 100                      │   │   │TX 100                      │
└─────────────┬──────────────┘   │   └─────────────┬──────────────┘   │   └─────────────┬──────────────┘
              │ HASH             │                 │ HASH             │                 │ HASH
┌─────────────▼──────────────┐   │   ┌─────────────▼──────────────┐   │   ┌─────────────▼──────────────┐
│0000000000000010001110101001├───┘   │0000000000000011100110101001├───┘   │0000000000000011100110101100│
└────────────────────────────┘       └────────────────────────────┘       └────────────────────────────┘
```

- Les utilisateurs normaux n'ont pas besoin d'avoir toute la chaine, ils peuvent se contenter de regarder les blocks qui les concernent
- Les mineurs ont intérêt à bien se comporter car :
  - si un block est invalide, la coinbase est invalide
  - Quand deux informations contradictoires sont reçus, celle qui a le plus de travail est acceptée
    - Donc si je reçois un block valide et que j'étais en train de chercher un nonce, j'ai intérêt à arrêter mes recherche et prendre ce block comme nouveau parent
- Si je veux rétroactivement modifier un block ou changer l'ordre des bloques, je dois trouver des nonce pour les bloques que je modifie et tous les bloques qui suivent

---
# Registre
## Solutions
### Délégation de la verification aux mineurs
#### BlockChain
##### Effets de bord
- les transactions ne sont pas directes
  - Avant j'envoyais les transactions aux utilisateurs et ils l'acceptaient
  - Maintenant les transactions sont en attentes d'être incluent dans un bloque et ne valent rien temps qu'elles ne sont pas minées
- La liste des bloques est stockées sur tous les ordinateurs de la même manière sauf en cas de fork
  - Une fork arrive quand deux utilisateur trouvent un bloque différent et le communique aux utilisateurs du registre en même temps
    - L'idée est que les forks se résolvent assez vite parce que les mineurs n'ont pas intérêt a essayer des nonce si leur bloque ne sera pas accepté par la majorité du registre. Donc quand un mineur découvre qu'il est en train de miner sur une fork qui est partagée par une minorité du registre, il a intérêt a se mettre a jour avec la majorité
- On peut demander un nombre plus ou moins grands de 0 au debut du hash pour moduler le temps de creation d'un block en fonction de la puissance de calcule allouée à créer des blocks
- On peut changer le nombre de Lifen coin créés a la coinbase

---
# Registre
On a inventé un registre 
- décentralisé (pas de banque qui valide ou non nos transactions)
- sans confiance (tous les membres ont intérêt à ce comporter honorablement (a peu pré))
- immuable
- publique
---
# Registre
## Désavantages
- Pollution
  - On pourrait utiliser la proof of stake au lieu de la proof of work
- Les LC ne valent rien parce que ils sont pas réels
  - Peut être qu'ils ne valent rien, mais ils ne sont pas plus irréels que n'importe quelle monnaie. Une monnaie ne fonctionne que temps que tout le monde accepte sa valeur. Si on perd la confiance, la monnaie n'a plus de valeur. C'est un peu ce qui arrive aux crypto en ce moment d'ailleurs.
- Débit
  - En fonction de la difficulté de la proof of work (nombre de 0) et du nombre de transaction par block, le debit peut être plus ou moins limité, mais visa sera probablement toujours plus efficace
  
---
# Refs
- facile (20min) : https://www.youtube.com/watch?v=bBC-nXj3Ng4
- mit (20h) : https://www.youtube.com/watch?v=EH6vE97qIP4&list=PLUl4u3cNGP63UUkfL0onkxF6MYgVa04Fn
- Satoshi's paper : https://bitcoin.org/bitcoin.pdf
  
  
  