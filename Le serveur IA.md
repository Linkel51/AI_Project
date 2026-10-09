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

## vLLM

Chaque vLLM est isolé dans un conteneur docker, c'est lui qui fait tourner le modèle de qwen 3.8 27b, il est spécialisé pour un agent IA, il gère nativement le multi-lora ce qui fait qu'un vLLM peut gérer plusieurs utilisateurs simultané utilisant chacun un lora (ARIA ou ARON) différents, le vLLM est optimisé pour l'IA et peut gérer plusieurs requêtes simultanées (grâce à une gestion matricielle des requêtes), il est capable de gérer des contextes très larges et de générer du code complexe.

## Les lora

Les lora sont des modèles spécialisés qui sont appliqués sur le modèle de base Qwen 3.8 27b, ils permettent de spécialiser le modèle pour un agent IA donné, les lora sont appliqués sur le modèle de base par le vLLM, chaque lora est spécialisé pour un agent IA donné (ARIA ou ARON), les lora modifient intrasèquement le comportement, la manière de réfléchir et les réponses du modèle IA.

## Le LiteLLM

LiteLLM est le routeur des requêtes et le controleur d'accès, c'est lui qui va aiguillé les requêtes accépté pour repartir la charge de travaille sur les modèles ia, il permet de gérer les 2 api par utilisateur (une par ia (ARON ET ARIA)) ce qui permet a l'aide d'api ou de script de bloquer une api si on ne veut plus qu'elle passe par exemple et de la réactiver plus tard si on veut qu'elle passe à nouveau.
