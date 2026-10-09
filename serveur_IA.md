# Le serveur IA

Ce document décrit la partie « calcul » : les GPU, le modèle, vLLM et LiteLLM. L'interface web est décrite dans [OpenWebUi.md](OpenWebUi.md), la recherche dans les cours dans [RAG.md](RAG.md).

## Architecture

```txt
+-----------------------------------------------------------+
|                          LiteLLM                          |
|  - modèles virtuels « aria » et « aron »                  |
|  - une clé API par étudiant et par IA                     |
|  - ajoute le prompt système puis répartit la charge       |
+-----------+-------------------+-------------------+-------+
            |                   |                   |
            v                   v                   v
     +-------------+     +-------------+     +-------------+
     |  Docker 1   |     |  Docker 2   |     |  Docker N   |
     |  vLLM       |     |  vLLM       |     |  vLLM       |  ...
     |  Qwen3.8 27B|     |  Qwen3.8 27B|     |  Qwen3.8 27B|
     |  AWQ 4 bits |     |  AWQ 4 bits |     |  AWQ 4 bits |
     |  GPU 24 Go  |     |  GPU 24 Go  |     |  GPU 24 Go  |
     +-------------+     +-------------+     +-------------+
```

 - Une machine virtuelle héberge l'ensemble.
 - Un conteneur Docker par GPU, chacun avec son vLLM et sa copie du modèle.
 - Un seul LiteLLM en façade. C'est le seul point d'entrée : OpenWebUI et Cline ne parlent jamais directement à vLLM.
 - Deux « modèles virtuels » dans LiteLLM, `aria` et `aron`, qui pointent vers les mêmes vLLM. Ce qui les distingue est le prompt système que LiteLLM ajoute.

## Le modèle et sa quantification

Le modèle de base est **Qwen3.8 27B** (licence Apache 2.0, contexte jusqu'à 262 144 tokens). En pleine précision il est trop gros pour une carte de 24 Go, il est donc utilisé en 4 bits :

| Format | Usage | Décision |
|---|---|---|
| **AWQ 4 bits (W4A16)** | vLLM, environ 16 à 19 Go de VRAM | **Retenu.** Pas de version officielle : les AWQ viennent de la communauté (par exemple `barrydeen/Qwen3.8-27B-AWQ-4bit`). Télécharger depuis un auteur fiable et lire la fiche du modèle. |
| GGUF Q4_K_M | llama.cpp, Ollama | Non retenu : mal supporté par vLLM. |

Avec environ 17 Go de poids sur 24 Go, il reste 5 à 6 Go pour le cache KV (la mémoire de travail des conversations). Conséquences :

 - il faut limiter la longueur du contexte avec `--max-model-len` ;
 - l'AWQ demande `--dtype float16` ;
 - on estime 2 à 4 sessions confortables par GPU. ARON (Cline) envoie de gros contextes et sera le facteur limitant, le RAG d'ARIA en ajoute aussi (voir [RAG.md](RAG.md)).

## vLLM

vLLM est le moteur qui exécute le modèle sur le GPU. Il traite plusieurs requêtes en même temps en les regroupant, et peut réutiliser le calcul d'un début de prompt identique d'une requête à l'autre (prefix caching, à vérifier dans la version utilisée). C'est utile ici : les prompts système d'ARIA et d'ARON sont les mêmes pour tout le monde.

Version visée : 0.17 ou plus récent (avec transformers 5.8 ou plus), d'après les guides trouvés. Il existe une [recette officielle vLLM](https://recipes.vllm.ai/Qwen/Qwen3.8-27B) pour ce modèle, mais elle couvre les variantes FP8 et NVFP4, pas l'AWQ.

## LiteLLM

LiteLLM est le routeur et le contrôleur d'accès. Il fait quatre choses :

1. **Authentifier** : chaque étudiant a deux clés API, une pour ARIA et une pour ARON. Une clé peut être bloquée puis réactivée sans toucher à l'autre.
2. **Ajouter le prompt système** de l'IA demandée (voir ci-dessous).
3. **Répartir la charge** entre les vLLM disponibles.
4. **Journaliser** les usages (qui, quand, combien).

### Deux comportements, un seul modèle

ARIA et ARON utilisent exactement le même modèle chargé sur les GPU. Leur différence vient uniquement du **prompt système** que LiteLLM ajoute à chaque requête, selon le modèle virtuel demandé (`aria` ou `aron`). Il n'y a aucun entraînement ni adaptateur supplémentaire : un prompt se modifie en quelques minutes, et chaque GPU peut servir indifféremment l'une ou l'autre IA.

### Comment le prompt est ajouté

LiteLLM propose des « hooks » qui modifient la requête juste avant son envoi au modèle (`async_pre_call_hook`, dans les [call hooks](https://docs.litellm.ai/docs/proxy/call_hooks)). Un petit module Python y ajoute le prompt selon le modèle demandé :

```python
from litellm.integrations.custom_logger import CustomLogger

PROMPTS = {"aria": "...", "aron": "..."}   # contenu : dossier prompts/

class InjectionPrompt(CustomLogger):
    async def async_pre_call_hook(self, user_api_key_dict, cache, data, call_type):
        prompt = PROMPTS.get(data.get("model"))
        if prompt and data.get("messages"):
            data["messages"].insert(0, {"role": "system", "content": prompt})
        return data

proxy_handler_instance = InjectionPrompt()
```

Ceci est un exemple adapté de la documentation, pas du code testé : la signature exacte dépend de la version de LiteLLM.

Avantages par rapport à un prompt réglé dans OpenWebUI :

 - il s'applique quel que soit le client (navigateur, Cline, script) ;
 - l'étudiant ne peut ni le voir, ni le retirer, ni le remplacer par le sien ;
 - on le met à jour à un seul endroit.

Points d'attention :

 - **ARON** : Cline envoie déjà son propre prompt système (format des outils, consignes). Le prompt d'ARON doit s'y **ajouter** sans le contredire. Il est donc ajouté en complément, et le cas où un message système existe déjà doit être géré (fusion plutôt que simple ajout).
 - **ARIA** : un second rappel court peut être ajouté **après** le dernier message de l'étudiant, car les modèles suivent mieux les consignes les plus récentes (voir [prompts/ARIA.md](prompts/ARIA.md)).

Le contenu des prompts est dans le dossier [prompts/](prompts/).

## Points à valider avant d'acheter le matériel

Ces points n'ont pas pu être confirmés par la documentation. Ils se testent sur **une seule** carte de 24 Go :

| # | Question | Pourquoi c'est important |
|---|---|---|
| 1 | vLLM démarre l'AWQ de Qwen3.8 27B sur un seul GPU, avec un `--max-model-len` utile | Les exemples trouvés utilisent 2 GPU (`--tensor-parallel-size 2`). Si un GPU ne suffit pas, il faut 2 GPU par instance et le nombre d'instances est divisé par deux. |
| 2 | Nombre de requêtes parallèles avant que la latence ne devienne gênante, avec Cline et avec le RAG | Dimensionne tout l'achat. |
| 3 | Le hook LiteLLM ajoute bien le prompt, sans conflit avec Cline ni avec OpenWebUI | Sans lui, ni ARIA ni ARON ne se comportent comme prévu. |
| 4 | ARIA résiste aux contournements (jeu de tests dans [prompts/ARIA.md](prompts/ARIA.md)) | C'est la raison d'être d'ARIA. |
