# Predictions-boursieres

Where will the next trade take place? : https://challengedata.ens.fr/participants/challenges/40/

L’objectif est de prédire sur quelle bourse (parmi six) la prochaine transaction aura lieu, en se basant sur les carnets d’ordres et les dix dernières transactions.

# Données
**Carnet d'Ordres (Order Book)**
Un carnet d'ordres est la liste des ordres passées par les investisseurs : 
  - bid : ordres d'achats
  - ask : ordres de vente

Pour chaque bourse on a: ask prices/volumes et bid prices/volumes aux 5 premiers niveaux.

**Les 10 dernières transactions** : timestamp, bourse (venue), prix, quantité.

## Création de colonnes
- _venue_frequence_10_ : La plateforme/source_id qui est apparue le plus souvent dans les 10 dernieres transactions
- _venue_frequence_5_ : La plateforme/source_id qui est apparue le plus souvent dans les 5 dernieres transactions
- _venue_frequence_3_ : La plateforme/source_id qui est apparue le plus souvent dans les 3 dernieres transactions
- _liquidity_bid_ de 0 à 5 
- _liquidity_ask_ de 0 à 5 
- _total_liquidity_ de 0 à 5 : liquidité totale de chaque bourse
- _venue_with_max_bidsize_
- _venue_with_max_asksize_
- _venue_with_best_bid_ : la bourse avec le bid le plus elevé
- _venue_with_best_ask_ : la bourse avec le ask le plus faible
- _matching_venue_ de 0 à 5 : 6 colonnes binaires qui indiquent si une bourse a un ask_size=bid_size.
- _venue_with_min_spread_
- _venue_with_min_spread1_

## Modèles testés
- LightGBM : modèle principal, bonne performance en local (score ~0.505) et un test final (score ~0.4947), qui nous place en 24ème position du challenge.
- CatBoost : testé, avec résultats similaires, mais un moins bon score final.
- XGBoost : résultats proches mais plus lent que un LightGBM
- Blending entre un LightGBM et un LightGBM avec pondération de poids : On remarque avec une _Matrice de Confusion_ que le modèle non pondéré est fort sur les classes fréquentes. Le modèle pondéré est plus fort sur les classes rares. Le blending combine les deux pour un compromis optimal. Permet d'avoir des prédictions plus équilibrées, il prédit plus souvent les classes les plus rares. 

## Quelques observations qui justifient le rajout des nouvelles features : 

- Plus un actif est **liquide**, on pourra l'acheter plus rapidement, en grande quantité et au prix que l'on demande
- Si un actif est **peu liquide**, il y a peu d'ordres en face, donc il est plus compliqué de le vendre, on risque de payer plus cher (ou de vendre à perte)
- Plus on a des volumes a differents niveaux de prix (_ask_size_ et _bid_size_), plus l'actif est liquide.
- Plus le **spread est faible**, plus il y a des la concurrence sur le marché **donc plus l'actif est liquide**
- On ne peut pas vendre plus cher que le ask = **Le BID est toujours inferieur ou egal au ASK**
- faut gerer les ordres, donc la quantité a une importance aussi .
- Le 'Trade Price' correspond à la difference entre le Prix du Trade (généralement = ask) et le aggregate mid-price
- Le 'Trade price' donne une idée de la préssion acheteur/vendeur : si souvent au-dessus, **les acheteurs sont frustrés.**

- Généralement on **veut éviter les marchés peu liquides** = **réduire le spread**

## Cross-validation & Evaluation
- Utilisation d’un train/test split
- Confusion matrix utilisée pour analyser les erreurs par classe.
- Pondération des classes via class_weight dans LightGBM testée (faible impact). 
- Importance des features analysée.


## Potentielles pistes d'amélioration : 

- rajouter le _avg_trade_price_ ou le _std_trade_price_
- faire des études sur les _ts_last_update_ : si le carnet change rapidement, c'est à dire que les clients passent des ordres en continu, c'est un bon signe de liquidité

**Pourquoi il est interessant d'etudier le comportement des individus/traders?**

Le comportoment/pressions sur les acheteurs et les vendeurs nous donnent des informations sur la direction du marché. 
Par exemple les _avg_trade_price_, _std_trade_price_ , capturent la volatilité récente et la diréction du marché: **ex:** si les derniers trades ont des prix élevés par rapport au mid-price, ça peut indiquer une pression acheteuse, et certaines plateformes sont peut-être plus favorables aux acheteurs.


