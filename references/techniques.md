# Techniques d'économie de tokens

Aide-mémoire concret. But : contexte ≈ 20 % de ce qu'il serait en
lecture naïve.

## 1. Lire moins, lire mieux

```text
# Chercher avant de lire (localise → lire le voisinage seulement)
grep -n "def ma_fonction" fichier.py          # + read offset/limit autour
rg "pattern" --glob '!node_modules' --glob '!dist'

# Structure d'un gros fichier sans le lire
rg -n "^(def |class |function |export |# )+" fichier
```

- Lecture naïve d'un fichier de 2 000 lignes ≈ 20–40k tokens.
- Grep + lecture d'un bloc de 40 lignes ≈ 500 tokens.
- **Gain : 95 %+ sur les fichiers exploratoires.**

## 2. Digests (cache de session)

Après chaque lecture utile, noter une ligne réutilisable :

```text
src/config.ts (148 l.) → port=8080, DB_URL via env, timeout=30s
package.json → react 19.0.1, scripts: dev/build/test
logs/app.log (erreur 1 seule) → ECONNREFUSED sur :5432 à 14:02
```

Règle : **si la ligne existe déjà dans la conversation, ne pas rouvrir
le fichier.** Re-lire seulement après modification réelle du fichier.

## 3. Condensation par type de fichier

### JSON / YAML (configs, manifestes)

```sh
jq '{name, version, scripts, dependencies: (.dependencies|keys)}' package.json
```

Ne garder que les clés qui servent à la tâche.

### Code

- Lire les **en-têtes et signatures** d'abord :
  `rg -n "^(export |def |class |function )" src/`
- Lire le corps d'une seule fonction, pas tout le module.

### CSV / logs

```sh
head -1 fichier.csv                      # en-têtes
wc -l fichier.csv                        # taille réelle
rg "ERROR|FAIL" fichier.log | tail -50   # erreurs seulement
```

### Bases / requêtes

- Schéma (`\d table`, `SHOW CREATE TABLE`) plutôt que `SELECT *`.

## 4. Digest structuré pour section longue

Quand un fichier est tronqué, marquer explicitement :

```markdown
## fichier.md (tronqué : lignes 1–120 sur 900)
- Sections : intro, API, config, FAQ (lignes 700–900 non lues)
- Faits retenus : …
```

Aucun fait inventé ; rien d'omis qui soit critique pour la tâche.

## 5. Anti-patrons (gaspillage classique)

| Gaspillage | Économie |
| --- | --- |
| Lire un fichier entier pour trouver un mot | grep + voisinage |
| Relire un fichier déjà lu | digest en conversation |
| `ls -R` sur un gros dépôt | glob ciblé ou grep |
| Copier une sortie d'outil complète dans la réponse | ne garder que le pertinent |
| Réexplorer après une édition | l'édition confirme le nouveau contenu |
| Lire `node_modules`, `dist`, lockfiles | jamais |
| Répondre en reformulant tout le contexte | réponse directe |

## 6. Vérification du bilan

En fin de tâche longue, demander (ou s'auto-demander) :
- [ ] Quels fichiers ont été lus **plus d'une fois** sans modification ?
- [ ] Quelles sorties d'outils ont été recopiées inutilement ?
- [ ] Un digest existe-t-il pour chaque découverte importante ?

Se combine avec `anti-dream` : l'économie ne doit jamais transformer un
fait vérifié en affirmation approximative.
