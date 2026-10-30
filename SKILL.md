---
name: Opti-Skill
description: Optimiser la consommation des tokens — garder en cache les données importantes pour éviter de relire les fichiers, lire de façon ciblée (grep avant read), et condenser les fichiers non-Markdown en digests avant lecture. À utiliser sur les gros dépôts, les tâches longues, ou quand l'utilisateur veut réduire le coût/latence d'une session.
---

# Opti-Skill : économie de tokens

Objectif : garder le contexte petit sans perdre d'information. Chaque
octet lu doit servir ; ce qui a déjà été lu ne se relit pas.

> Limite honnête : ce skill ne contrôle pas le cache serveur du
> fournisseur de modèles. Il agit sur ce qui est vraiment contrôlable :
> la taille du contexte envoyé à chaque étape.

## 1. Cache de session (ne pas relire)

- **Une lecture = une trace.** Après avoir lu un fichier, enregistrer
  dans la conversation un **digest d'une ligne** :
  `chemin → taille, rôle, faits clés (constantes, signatures, adresses)`.
  Les étapes suivantes s'appuient sur le digest, pas sur une relecture.
- **Jamais de relecture identique.** Si une information est déjà dans le
  contexte (contenu lu, sortie d'outil), la réutiliser telle quelle.
  Re-lire uniquement si le fichier a pu changer (après édition, build,
  `git pull`).
- **Digest persistant pour les tâches longues** : si la session risque
  de dépasser quelques étapes lourdes, maintenir un mémo compact
  (résumé des découvertes : chemins, blocages, décisions) plutôt que de
  rouvrir les fichiers pour retrouver le même fait.
- **Mémoire des sorties d'outils** : les listings, résultats de grep et
  tests déjà exécutés sont réutilisés ; ne pas relancer une commande
  identique sans raison.

## 2. Lecture ciblée (grep avant read)

- **Chercher avant de lire** : `grep` / `rg` localise la ligne → lire
  seulement le voisinage (`offset`/`limit`), jamais tout le fichier.
- **Lire par sections** : sur un fichier > ~300 lignes, lire d'abord la
  structure (grep des définitions, en-têtes), puis les blocs utiles.
- **`glob`/`grep` d'abord** : ne pas lister un répertoire complet quand
  un motif suffit.
- **Sauter le bruit** : `node_modules`, `dist`, verrous de paquets,
  journaux, minifiés, fichiers générés — jamais lus en entier.
- **Éditions ciblées** : privilégier l'outil d'édition avec une
  `oldString` précise plutôt que read → réécriture complète.

## 3. Condensation Markdown des fichiers non-Markdown

Avant d'ingérer un gros fichier non-Markdown, en extraire un **digest
Markdown minimal** au lieu du contenu complet :

| Type | Méthode | Digest |
| --- | --- | --- |
| JSON / YAML (config, manifestes) | extraire **seules les clés utiles** (`jq`, lecture ciblée) | tableau clé → valeur pertinente |
| Code source | grep des définitions + blocs ciblés | liste des exports/fonctions/segments |
| CSV / logs | colonnes d'intérêt, agrégats, dernières lignes | résumé + exemples significatifs |
| Base de données / requêtes | schéma + requêtes pertinentes | liste des tables/champs utilisés |
| Fichiers très longs | sections relatives à la tâche | résumé structuré par section |

Règles :
- **Ne jamais convertir un fichier entier** pour « faire du Markdown »
  — cela ferait *gagner* du contexte seulement si le digest est plus
  petit. Objectif : digest ≤ 20 % de la taille source.
- **Fidélité** : un digest ne doit jamais inventer ni omettre un fait
  qui comptera plus tard ; noter *« tronqué »* si pertinent.
- Un fichier déjà Markdown ou petit (< ~100 lignes) est lu directement.

## 4. Règles d'économie générale

- **Batch** : regrouper les appels indépendants en parallèle plutôt
  que séquentiellement (moins d'allers-retours, réponses fusionnées).
- **Pas d'exploration gratuite** : ne lire un fichier que s'il sert à
  la tâche en cours.
- **Résumés de sortie** : quand un outil renvoie une sortie massive,
  n'en retenir que les lignes pertinentes, pas tout recopier.
- **Réponses compactes** : pas de re-listing de ce qui est déjà connu,
  pas de répétition du contenu des fichiers cités.

## 5. Déclenchement

Appliquer en priorité sur : gros dépôts, sessions longues, lectures
répétées des mêmes fichiers, configs/code volumineux, quand
l'utilisateur mentionne coût, tokens, latence ou contexte. Se combine
avec `anti-dream` (vérification) et `auto-chose` (choix du skill).
