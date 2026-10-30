# Opti-Skill

> Skill OpenCode d'optimisation de la consommation de tokens — cache de session, lecture ciblée, condensation des fichiers en digests Markdown.

**ID** : `opti-skill` · **Portée** : globale (`~/.config/opencode/skills/`) · **Langue** : FR

## Ce que fait le skill

Garde le contexte petit sans perdre d'information :

1. **Cache de session** : une lecture = un digest d'une ligne ; jamais de relecture identique sauf si le fichier a changé
2. **Lecture ciblée** : `grep` avant `read`, lectures par sections, skip de `node_modules`/`dist`
3. **Condensation Markdown** : JSON → `jq` sur les clés utiles, code → signatures, logs → erreurs ; objectif digest ≤ 20 % de la source
4. **Règles générales** : batch d'appels parallèles, pas d'exploration gratuite, réponses compactes

> Limite honnête intégrée : le cache serveur du fournisseur de modèles n'est pas contrôlable — le skill agit sur la taille réelle du contexte envoyé.

## Structure

```text
opti-skill/
├── SKILL.md                      # 5 sections de règles d'économie
└── references/
    └── techniques.md             # Commandes concrètes, exemples de digests, anti-patrons
```

## Déclenchement

- **Automatique** : gros dépôts, sessions longues, mentions de coût/tokens/contexte
- **Manuel** : `@opti-skill`
- Gain indicatif : ~95 % sur les fichiers exploratoires (grep + bloc vs lecture entière)

## Vérification

- [ ] Aucun fichier lu deux fois sans modification
- [ ] Aucune sortie d'outil recopiée inutilement
- [ ] Digest existant pour chaque découverte importante

## Skills liés

- `anti-dream` (l'économie ne dégrade jamais la fiabilité), `session-handoff` (digests de reprise)
