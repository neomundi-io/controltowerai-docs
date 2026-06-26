# ControlTowerAI — Guide d’intégration des providers LLM

> **Pour qui ?** Développeurs, équipes produit et intégrateurs qui souhaitent faire passer leurs requêtes LLM par ControlTowerAI.
> **Endpoint principal :** `POST https://api.neomundi.io/v1/govern/stream`
> **Statut :** Documentation publique d’intégration.

ControlTowerAI permet d’appeler un modèle de langage via une couche de mesure et de gouvernance runtime.

L’objectif est simple : vous continuez à utiliser votre provider et votre modèle habituel, tandis que ControlTowerAI vous aide à rendre l’exécution plus observable, traçable et exploitable en production.

---

## Table des matières

1. [Démarrage rapide](#1-démarrage-rapide)
2. [Les deux clés à utiliser](#2-les-deux-clés-à-utiliser)
3. [Envoyer une requête](#3-envoyer-une-requête)
4. [Choisir un provider et un modèle](#4-choisir-un-provider-et-un-modèle)
5. [Providers actuellement pris en charge](#5-providers-actuellement-pris-en-charge)
6. [Réponses temps réel avec SSE](#6-réponses-temps-réel-avec-sse)
7. [Erreurs fréquentes et solutions](#7-erreurs-fréquentes-et-solutions)
8. [Bonnes pratiques de sécurité](#8-bonnes-pratiques-de-sécurité)
9. [Exemples d’intégration](#9-exemples-dintégration)

---

# 1. Démarrage rapide

Pour effectuer votre première requête, vous avez besoin de :

1. Une clé API ControlTowerAI ;
2. Une clé API valide chez votre provider LLM ;
3. Le nom du provider ;
4. Le nom du modèle que vous souhaitez utiliser.

La recommandation la plus fiable est de toujours renseigner explicitement le champ `provider`.

Exemple minimal avec OpenAI :

```bash
curl -N https://api.neomundi.io/v1/govern/stream \
  -H "X-API-Key: ct_live_xxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Quelle est la capitale de la France ?",
    "provider": "openai",
    "provider_api_key": "sk-proj-xxxxxxxx",
    "model": "gpt-4o-mini"
  }'
```

---

# 2. Les deux clés à utiliser

Une requête ControlTowerAI utilise deux clés distinctes.

| Clé                 | Emplacement                      | Rôle                                                   |
| ------------------- | -------------------------------- | ------------------------------------------------------ |
| Clé ControlTowerAI  | Header `X-API-Key`               | Authentifie votre application auprès de ControlTowerAI |
| Clé du provider LLM | Champ `provider_api_key` du body | Permet d’appeler le modèle LLM de votre choix          |

Exemple :

```text
X-API-Key: ct_live_xxxxxxxxx
```

```json
{
  "provider_api_key": "sk-proj-xxxxxxxx"
}
```

> Ne confondez pas les deux clés.
> Une clé ControlTowerAI ne doit jamais être utilisée comme clé provider, et une clé provider ne doit jamais être utilisée dans le header `X-API-Key`.

---

# 3. Envoyer une requête

## Endpoint

```text
POST https://api.neomundi.io/v1/govern/stream
```

L’endpoint utilise le streaming SSE afin que votre application puisse recevoir la réponse au fil de la génération.

## Headers requis

```http
X-API-Key: ct_live_xxxxxxxxx
Content-Type: application/json
```

## Champs du body

| Champ              |     Requis | Valeur par défaut              | Description                                                             |
| ------------------ | ---------: | ------------------------------ | ----------------------------------------------------------------------- |
| `prompt`           |        Oui | —                              | Question ou instruction utilisateur à envoyer au modèle                 |
| `provider`         | Recommandé | —                              | Provider LLM à utiliser, par exemple `openai`, `anthropic` ou `mistral` |
| `provider_api_key` |        Oui | —                              | Clé API associée au provider choisi                                     |
| `model`            | Recommandé | Variable                       | Identifiant du modèle à appeler chez ce provider                        |
| `system_prompt`    |        Non | `You are a helpful assistant.` | Instruction système envoyée au modèle                                   |
| `max_tokens`       |        Non | `4096`                         | Nombre maximal de tokens générés                                        |
| `temperature`      |        Non | `0.7`                          | Niveau de variabilité de la génération, de `0.0` à `2.0`                |

## Exemple de body complet

```json
{
  "prompt": "Explique la photosynthèse simplement, en trois paragraphes.",
  "provider": "openai",
  "provider_api_key": "sk-proj-xxxxxxxx",
  "model": "gpt-4o-mini",
  "system_prompt": "You are a precise and accessible science educator.",
  "max_tokens": 800,
  "temperature": 0.4
}
```

---

# 4. Choisir un provider et un modèle

ControlTowerAI sépare clairement :

* le **provider**, c’est-à-dire l’entreprise ou l’infrastructure qui exécute le modèle ;
* le **model**, c’est-à-dire le modèle précis que vous souhaitez appeler.

Exemples :

| Provider     | Modèle possible            |
| ------------ | -------------------------- |
| `openai`     | `gpt-4o-mini`              |
| `anthropic`  | `claude-3-5-sonnet-latest` |
| `google`     | `gemini-2.0-flash`         |
| `mistral`    | `mistral-large-latest`     |
| `perplexity` | `sonar`                    |
| `deepseek`   | `deepseek-chat`            |
| `qwen`       | `qwen-plus`                |

Utilisez toujours un nom de modèle autorisé par votre compte auprès du provider concerné.

> ControlTowerAI ne remplace pas votre compte provider. Vous devez disposer de vos propres droits d’accès, clés API et autorisations d’usage chez le provider choisi.

---

# 5. Providers actuellement pris en charge

| Valeur `provider` | Exemples de modèles                              |
| ----------------- | ------------------------------------------------ |
| `openai`          | `gpt-4o`, `gpt-4o-mini`                          |
| `anthropic`       | `claude-3-5-sonnet-latest`                       |
| `google`          | `gemini-2.0-flash`, `gemini-2.5-pro`             |
| `mistral`         | `mistral-large-latest`                           |
| `infomaniak`      | `llama3.1`                                       |
| `xai`             | `grok-4`                                         |
| `deepseek`        | `deepseek-chat`                                  |
| `perplexity`      | `sonar`, `sonar-pro`                             |
| `cohere`          | `command-r-plus`                                 |
| `moonshot`        | `moonshot-v1-8k`                                 |
| `yi`              | `yi-large`                                       |
| `baidu`           | `ernie-4.0-8k`                                   |
| `qwen`            | `qwen-plus`, `qwen-max`                          |
| `groq`            | `llama-3.3-70b-versatile`                        |
| `together`        | `meta-llama/Llama-3.1-70B-Instruct-Turbo`        |
| `meta`            | Selon les modèles disponibles via votre endpoint |
| `apertus`         | `apertus-8b`                                     |
| `microsoft`       | Configuration Azure OpenAI spécifique requise    |

> La disponibilité exacte des modèles dépend du provider, de votre région, de votre contrat et des droits associés à votre clé API.

---

# 6. Réponses temps réel avec SSE

L’endpoint `/v1/govern/stream` retourne un flux SSE.

Cela permet à votre application de recevoir progressivement :

* les éléments de génération ;
* les événements de traitement ;
* les informations utiles à l’observation runtime ;
* la réponse finale.

Avec `curl`, l’option `-N` permet de désactiver le buffering et d’afficher les événements au fil de l’eau :

```bash
curl -N https://api.neomundi.io/v1/govern/stream \
  -H "X-API-Key: ct_live_xxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Résume les trois causes principales du changement climatique.",
    "provider": "mistral",
    "provider_api_key": "xxxxxxxx",
    "model": "mistral-large-latest"
  }'
```

Dans une application web, utilisez `fetch()` avec lecture du flux ou un client SSE compatible avec votre environnement.

---

# 7. Erreurs fréquentes et solutions

## Erreur : clé ControlTowerAI invalide

Vérifiez que votre header contient bien :

```http
X-API-Key: ct_live_xxxxxxxxx
```

Ne placez pas cette clé dans `provider_api_key`.

---

## Erreur : clé provider invalide

Vérifiez que :

* votre clé appartient bien au provider déclaré ;
* elle est toujours active ;
* votre compte provider autorise le modèle demandé ;
* elle n’a pas été copiée avec des espaces ou caractères supplémentaires.

---

## Erreur : provider non reconnu

Utilisez une valeur présente dans la liste des providers pris en charge.

Exemple correct :

```json
{
  "provider": "mistral"
}
```

Exemple incorrect :

```json
{
  "provider": "Mistral AI"
}
```

Les valeurs de provider doivent être écrites en minuscules et correspondre exactement à la documentation.

---

## Erreur : modèle non disponible

Le modèle demandé peut ne pas être actif sur votre compte provider.

Vérifiez :

1. Le nom exact du modèle ;
2. Les droits associés à votre clé ;
3. Les limites régionales ou contractuelles du provider ;
4. Les éventuelles règles de dépréciation du modèle.

---

## Erreur : `max_tokens` trop élevé

Tous les providers n’acceptent pas le même volume maximal de tokens.

Commencez avec une valeur raisonnable :

```json
{
  "max_tokens": 1024
}
```

Puis augmentez progressivement selon les capacités du modèle et votre cas d’usage.

---

## Erreur : réponse trop variable

Réduisez `temperature`.

Exemple pour une réponse plus déterministe :

```json
{
  "temperature": 0.2
}
```

Exemple pour une réponse plus créative :

```json
{
  "temperature": 0.9
}
```

---

# 8. Bonnes pratiques de sécurité

## Ne mettez jamais vos clés dans votre frontend public

Votre clé ControlTowerAI et votre clé provider doivent être manipulées côté serveur ou dans un environnement sécurisé.

Évitez de les exposer dans :

* le code JavaScript distribué au navigateur ;
* un dépôt Git public ;
* une capture d’écran ;
* un fichier `.env` versionné ;
* un ticket support partagé publiquement.

## Utilisez des variables d’environnement

Exemple :

```bash
export CONTROLTOWER_API_KEY="ct_live_xxxxxxxxx"
export OPENAI_API_KEY="sk-proj-xxxxxxxx"
```

Puis utilisez-les côté serveur :

```python
controltower_key = os.environ["CONTROLTOWER_API_KEY"]
provider_key = os.environ["OPENAI_API_KEY"]
```

## Utilisez des clés à privilèges limités lorsque votre provider le permet

Lorsque cela est disponible, privilégiez :

* une clé dédiée à votre environnement de production ;
* une clé distincte pour les tests ;
* des limites de dépense ;
* des restrictions d’usage ;
* une rotation régulière des clés.

---

# 9. Exemples d’intégration

## OpenAI

```bash
curl -N https://api.neomundi.io/v1/govern/stream \
  -H "X-API-Key: ct_live_xxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Rédige une synthèse de 150 mots sur les bénéfices de l IA dans la santé.",
    "provider": "openai",
    "provider_api_key": "sk-proj-xxxxxxxx",
    "model": "gpt-4o-mini",
    "temperature": 0.4,
    "max_tokens": 400
  }'
```

---

## Anthropic

```bash
curl -N https://api.neomundi.io/v1/govern/stream \
  -H "X-API-Key: ct_live_xxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explique les principaux risques d une décision automatisée dans un contexte bancaire.",
    "provider": "anthropic",
    "provider_api_key": "sk-ant-xxxxxxxx",
    "model": "claude-3-5-sonnet-latest",
    "temperature": 0.3,
    "max_tokens": 800
  }'
```

---

## Mistral

```bash
curl -N https://api.neomundi.io/v1/govern/stream \
  -H "X-API-Key: ct_live_xxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Propose un plan de déploiement pour un assistant IA interne.",
    "provider": "mistral",
    "provider_api_key": "xxxxxxxx",
    "model": "mistral-large-latest",
    "temperature": 0.5,
    "max_tokens": 1000
  }'
```

---

## Perplexity

```bash
curl -N https://api.neomundi.io/v1/govern/stream \
  -H "X-API-Key: ct_live_xxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Quels sont les principaux enjeux actuels de la souveraineté numérique européenne ?",
    "provider": "perplexity",
    "provider_api_key": "pplx-xxxxxxxx",
    "model": "sonar-pro",
    "temperature": 0.2,
    "max_tokens": 1000
  }'
```

---

# Besoin d’aide ?

Avant de contacter le support, préparez les éléments suivants :

* le provider utilisé ;
* le modèle demandé ;
* le statut HTTP reçu ;
* le message d’erreur exact ;
* un exemple de requête sans aucune clé réelle ;
* la date et l’heure approximatives de l’appel.

Ne partagez jamais une clé API réelle dans un message, une capture d’écran ou un ticket support.
