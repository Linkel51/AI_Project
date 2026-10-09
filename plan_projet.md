# Plan du projet : étapes, budget, risques

Ce document sert à présenter le projet et à décider. **Tous les montants sont des fourchettes d'estimation**, à remplacer par de vrais devis avant toute commande. Quand un prix vient d'une source, le lien est donné ; sinon c'est une estimation personnelle, signalée comme telle.

## 1. Étapes

| Étape | Contenu | Matériel nécessaire | Résultat attendu |
|---|---|---|---|
| **0. Prototype** | vLLM + AWQ sur une carte de 24 Go, LiteLLM, OpenWebUI, prompt d'ARIA, RAG sur 2 ou 3 cours | Une carte 24 Go (déjà disponible ou prêtée) | Les points de « À valider » ([serveur_IA.md](serveur_IA.md)) sont tranchés |
| **1. Pilote** | Un professeur et quelques étudiants volontaires testent ARIA et ARON ; mesure de la charge réelle | La même machine | Chiffres réels de capacité et retours des professeurs |
| **2. Portail d'accès** | Blocage par personne, groupe, salle ; lien avec l'emploi du temps | VM existante de l'IUT | Blocage d'un examen de bout en bout |
| **3. Achat et montée en charge** | Commande du matériel dimensionné d'après le pilote | Voir budget | Une promotion complète peut utiliser les IA |
| **4. Exploitation** | Surveillance, sauvegardes, mises à jour, bilan de l'année | — | Service stable |

**L'achat vient après le pilote.** Le dimensionnement repose aujourd'hui sur des estimations ; le pilote les remplace par des mesures et évite d'acheter trop ou trop peu.

## 2. Budget du matériel

### Prix de référence trouvés

| Élément | Prix constaté | Source / remarque |
|---|---|---|
| RTX 4090 24 Go, neuve, France | environ 2 200 à 3 000 € et plus selon le modèle | [Tech Insider](https://tech-insider.org/fr/?p=3809), [boutique NVIDIA France](https://marketplace.nvidia.com/fr-fr/consumer/graphics-cards/geforce-rtx-4090-aorus-master-24-go). Offre neuve en raréfaction, pas de date de relevé précise |
| RTX 4090, occasion | environ 1 500 à 2 500 $ aux États-Unis | [Tech Insider](https://tech-insider.org/rtx-4090-vs-rtx-5080-used-market-2026/), chiffres peu cohérents entre eux |
| RTX 6000 Ada 48 Go, neuve | environ 7 200 à 10 300 € en Europe | [GPUPrix](https://gpuprix.com/france/gpus/rtx-6000-ada-generation) (aucune offre en France relevée), [Serverschmiede](https://www.serverschmiede.com/en/nvidia-rtx-6000-ada-48gb-gddr6-pcie-40-x16-high-end-cad-server-workstation-grafikkarten-gpu-4x-display-port-new-rtx6000) |
| RTX 5090 32 Go | très au-dessus du prix de lancement, stock tendu | [Tom's Guide](https://www.tomsguide.com/computing/gpus/rtx-5090-gpu-prices-are-officially-out-of-control-now-usd1-400-over-nvidias-official-asking-price) : pas de prix français daté |

### Option A : 8 cartes de 24 Go (RTX 4090)

| Poste | Fourchette | Remarque |
|---|---|---|
| 8 × RTX 4090 | 17 600 à 24 000 € | Prix ci-dessus ; occasion possible mais sans garantie |
| 2 serveurs de 4 GPU (châssis, processeur, 128 Go de RAM, alimentations) | 6 000 à 12 000 € | **Estimation personnelle**, à confirmer par devis |
| 1 carte pour l'embedding et le reranker (8 à 12 Go) | 300 à 500 € | Estimation personnelle ; voir [RAG.md](RAG.md) |
| Stockage SSD, câblage, onduleur | 500 à 1 500 € | Estimation personnelle |
| **Total** | **environ 24 500 à 38 000 €** | |

L'estimation de départ du README (« environ 20 000 € ») était donc trop basse.

Points à vérifier : 8 RTX 4090 ne tiennent pas dans un seul serveur classique (encombrement, alimentation, refroidissement), d'où deux machines. La licence des cartes grand public peut en interdire l'usage en datacenter (d'après mes connaissances) : à vérifier avant achat.

### Option B : 2 cartes de 48 Go (RTX 6000 Ada)

| Poste | Fourchette | Remarque |
|---|---|---|
| 2 × RTX 6000 Ada | 15 000 à 20 600 € | Prix ci-dessus |
| 1 serveur de 2 GPU | 3 000 à 6 000 € | Estimation personnelle |
| 1 carte pour l'embedding et le reranker | 300 à 500 € | Estimation personnelle |
| Stockage, câblage, onduleur | 500 à 1 500 € | Estimation personnelle |
| **Total** | **environ 19 000 à 29 000 €** | |

Avantages : une seule machine, bien moins de consommation, et chaque carte laisse beaucoup plus de mémoire au cache des conversations que les 5 à 6 Go d'une carte de 24 Go. Inconvénient : moins d'instances en parallèle, donc moins de tolérance à la panne d'une carte.

### Option C : prototype

Si une carte de 24 Go est déjà disponible, **le prototype ne coûte pratiquement rien en matériel**. C'est l'étape à faire en premier.

### Une alerte sur la mémoire de 24 Go

Un banc d'essai de [Spheron](https://www.spheron.network/blog/rtx-5090-vs-rtx-4090/) classe un Qwen3 32B en 4 bits comme « marginal » sur une RTX 4090 (mémoire insuffisante avec le contexte par défaut). Qwen3.8 27B est plus petit, mais la marge reste étroite : c'est précisément ce que le test du prototype doit trancher. Si une carte de 24 Go ne suffit pas, l'option B devient plus intéressante.

### Pour comparer : la location de GPU

Louer des cartes à l'heure (environ 0,53 €/h pour une RTX 4090 d'après [Spheron](https://www.spheron.network/blog/rtx-5090-vs-rtx-4090/), tarif en dollars non converti) donnerait, pour 8 cartes, environ 9 000 € par an en usage de 10 h par jour pendant 220 jours. C'est un ordre de grandeur, pas une offre. Cette solution sort aussi les données de l'établissement, ce qui va à l'encontre d'une idée du projet : garder les conversations sur place.

## 3. Coûts de fonctionnement

| Poste | Option A | Option B |
|---|---|---|
| Puissance maximale estimée | environ 4,6 kW (8 × 450 W + 2 serveurs) | environ 0,9 kW (2 × 300 W + serveur) |
| Électricité par an (hypothèses : 10 h/jour à 50 % de charge, 220 jours, 0,20 €/kWh) | environ 1 000 € | environ 200 € |
| Refroidissement | à prévoir (salle ventilée ou climatisée) | plus simple |

Les puissances par carte viennent de mes connaissances et non d'une source citée ; le prix de l'électricité est une hypothèse (à remplacer par le tarif de l'IUT). Si les machines restent allumées toute l'année, la consommation à vide s'ajoute.

**Logiciels : 0 €.** vLLM, OpenWebUI, LiteLLM et Qwen3.8 (Apache 2.0) sont gratuits. Certaines fonctions avancées de LiteLLM (authentification unique, journalisation avancée) sont en offre payante d'après mes connaissances : à vérifier si besoin.

**Hébergement existant.** Le BUT possède déjà des serveurs de virtualisation (voir [Contexte.md](Contexte.md)) : LiteLLM, OpenWebUI et le portail peuvent y tourner dans des machines virtuelles, sans achat supplémentaire. Seuls les serveurs à GPU sont à acheter.

## 4. Risques

| Risque | Gravité | Réponse prévue |
|---|---|---|
| Une carte de 24 Go ne suffit pas pour le modèle en usage réel | Élevée | Test du prototype avant achat ; option B en secours |
| La capacité réelle est inférieure à l'estimation (Cline consomme beaucoup de contexte) | Élevée | Test de charge ; limite de débit par clé ; file d'attente |
| ARIA donne la solution malgré le prompt | Élevée | Jeu de tests ([prompts/ARIA.md](prompts/ARIA.md)), corrigés exclus de la base, vérificateur si besoin |
| Prix des cartes qui bougent d'ici la commande | Moyenne | Devis datés ; décision après le pilote |
| Panne d'une carte ou de LiteLLM | Moyenne | Plusieurs instances ; procédure de blocage manuel en examen ; sauvegardes |
| Données personnelles des étudiants | Moyenne | Voir ci-dessous |
| Le projet repose sur une seule personne | Moyenne | Documentation à jour, un référent côté IUT |

## 5. Données personnelles (RGPD)

Les conversations sont stockées et reliées à un étudiant identifié. À décider avec l'IUT avant le pilote :

 - qui peut lire les conversations (personne, professeur, administrateur) ;
 - combien de temps elles sont conservées, et comment un étudiant les supprime ;
 - l'information donnée aux étudiants sur ce qui est enregistré ;
 - la base légale et la déclaration auprès du délégué à la protection des données de l'établissement.

## 6. Ce qu'il faut demander à l'IUT

 - Où seront installés les serveurs (salle, alimentation électrique, refroidissement) et qui paie l'électricité.
 - Si la licence des cartes grand public est un problème pour l'établissement.
 - La source de l'emploi du temps pour le portail ([site_acces_IA.md](site_acces_IA.md)).
 - Qui valide le budget et à quelle échéance, et si un achat HT ou TTC est applicable.
 - Le référent côté IUT pour l'exploitation après la fin du projet.
