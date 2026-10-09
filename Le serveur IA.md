# LE SERVEUR IA

## STRUCTURE GLOBALE 

```txt
+---------------+ +---------------+ +---------------+
|    GPU 24Go   | |    GPU 24Go   | |    GPU 24Go   | ...
|  Qwen 3.8 27b | |  Qwen 3.8 27b | |  Qwen 3.8 27b |
+---------------+ +---------------+ +---------------+
        |                 |                 |
+---------------+ +---------------+ +---------------+
|      vLLM     | |      vLLM     | |      vLLM     | ...
+---------------+ +---------------+ +---------------+
    |       |         |       |         |       |
+------+ +------+ +------+ +------+ +------+ +------+
| lora | | lora | | lora | | lora | | lora | | lora | ...
| ARIA | | ARON | | ARIA | | ARON | | ARIA | | ARON |
+------+ +------+ +------+ +------+ +------+ +------+
    |       |         |       |         |       |
+---------------------------------------------------+
|                      liteLLM                      |
+---------------------------------------------------+
                          |
                    +-----------+
                    |   liens   |
                    +-----------+
```
 - 1 machine virtuelle
 - 1 docker par GPU
 - 1 vLLM avec qwen 3.8 27b par docker
 - 2 loRA spécialisé par vLLM (un prof socratique et un agent ia professionnel surpuissant)
 - 1 liteLLM qui fait la liaison entre les vLLM et le reste de l'infra

## Le modèle et sa quantification

Le modèle de base est Qwen3.8 27B (Apache 2.0, contexte jusqu'à 262 144 tokens). En pleine précision il est trop gros pour une carte de 24 Go, il est donc utilisé quantifié en 4 bits :

 - **AWQ 4 bits (W4A16)** : format retenu pour vLLM, environ 16 à 19 Go de VRAM, soit le même ordre de grandeur que le Q4_K_M. Il n'existe pas de version officielle, les AWQ viennent de la communauté (exemple : `barrydeen/Qwen3.8-27B-AWQ-4bit`, créé pour des cartes de 24 Go). Il faut télécharger depuis un auteur fiable et lire la fiche du modèle.
 - **GGUF Q4_K_M** : fait pour llama.cpp / Ollama, mal supporté par vLLM, donc non retenu ici.

Avec 17 Go de poids sur 24 Go, il reste environ 5 à 6 Go pour le cache KV. Il faut donc limiter la longueur de contexte (`--max-model-len`) et estimer 2 à 4 sessions confortables par GPU. ARON (Cline) envoie de gros contextes et sera le facteur limitant.

## vLLM

Chaque vLLM est isolé dans un conteneur docker, c'est lui qui fait tourner le modèle de qwen 3.8 27b, il est spécialisé pour un agent IA, il gère nativement le multi-lora ce qui fait qu'un vLLM peut gérer plusieurs utilisateurs simultané utilisant chacun un lora (ARIA ou ARON) différents, le vLLM est optimisé pour l'IA et peut gérer plusieurs requêtes simultanées (grâce à une gestion matricielle des requêtes), il est capable de gérer des contextes très larges et de générer du code complexe.

## Les lora

Les lora sont des modèles spécialisés qui sont appliqués sur le modèle de base Qwen 3.8 27b, ils permettent de spécialiser le modèle pour un agent IA donné, les lora sont appliqués sur le modèle de base par le vLLM, chaque lora est spécialisé pour un agent IA donné (ARIA ou ARON), les lora modifient intrasèquement le comportement, la manière de réfléchir et les réponses du modèle IA.

## Le LiteLLM

LiteLLM est le routeur des requêtes et le controleur d'accès, c'est lui qui va aiguillé les requêtes accépté pour repartir la charge de travaille sur les modèles ia, il permet de gérer les 2 api par utilisateur (une par ia (ARON ET ARIA)) ce qui permet a l'aide d'api ou de script de bloquer une api si on ne veut plus qu'elle passe par exemple et de la réactiver plus tard si on veut qu'elle passe à nouveau.

## Points à valider avant d'acheter le matériel

Ces points n'ont pas pu être confirmés par la documentation et doivent être testés sur une seule carte 24 Go :

 1. vLLM version 0.17 ou plus récent (avec transformers 5.8 ou plus) démarre l'AWQ sur **un seul GPU** avec un `--max-model-len` réduit. Les exemples trouvés utilisent 2 GPU (`--tensor-parallel-size 2`). Si un GPU ne suffit pas, il faudra 2 GPU par instance et le nombre d'instances sera divisé par deux.
 2. Le multi-LoRA (`--enable-lora`) fonctionne avec Qwen3.8 27B. Aucun test n'a été trouvé. SGLang est une alternative si vLLM pose problème.
 3. Les LoRA trouvés sur Hugging Face sont compatibles avec Qwen3.8 27B (même modèle de base) et leur licence est compatible avec l'usage en établissement.
 4. Test de charge : nombre de requêtes parallèles supportées avant dégradation de la latence.
