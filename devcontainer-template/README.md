# Template devcontainer sécurisé

Un point de départ pour isoler **chaque projet** dans son propre conteneur Docker,
avec Claude Code installé **dedans**. Objectif : si l'agent lance une bêtise, le
rayon de dégâts se limite au conteneur + au seul dossier projet monté — jamais le
reste de ta machine ni tes autres projets.

## Créer un projet sécurisé en 3 étapes

1. **Copier** ce dossier dans ton projet, renommé `.devcontainer` :
   ```bash
   cp -r ~/code/trimegix/dotfiles/devcontainer-template ~/code/trimegix/mon-projet/.devcontainer
   ```

2. **Éditer 4 repères** (cherche `CHANGE-ME` et les `👉` dans les fichiers) :
   | Fichier | Quoi changer |
   |---|---|
   | `devcontainer.json` | `name`, le volume `CHANGE-ME-claude` → `mon-projet-claude`, le `postCreateCommand`, les ports |
   | `Dockerfile` | la ligne `FROM` (couche STACK) si tu n'es pas en Node |

3. **Ouvrir** dans VS Code : `Ctrl+Shift+P` → **Dev Containers: Reopen in Container**.
   Au 1er lancement, tape `claude` et connecte-toi une fois. L'auth persiste ensuite.

## Ce que tu changes vs ce que tu ne touches pas

Le `Dockerfile` a deux couches :
- **COUCHE STACK** (le haut, `FROM …`) → la seule que tu adaptes par projet.
- **COUCHE SÉCURITÉ** (le bas) → identique partout, ne pas toucher.

## Les 3 pièges déjà résolus dans ce template

1. **Claude s'installe EN TANT QUE `node`, pas root.** Le paquet npm global échoue
   silencieusement en build : son postinstall télécharge un binaire natif et, lancé
   en root, le range dans le home de root — introuvable pour l'user `node`. On utilise
   donc l'installeur natif après `USER node`.
2. **`~/.claude` doit appartenir à `node`.** L'installeur y écrit `downloads/`. Sans le
   `chown -R node:node`, le build casse sur un « Permission denied ».
3. **L'auth se perd à chaque rebuild** sans le volume nommé. Le `mounts` de
   `devcontainer.json` la rend persistante (1 login par projet, puis c'est retenu).

## Adapter à une autre stack que Node

Les images `node:*` fournissent déjà un utilisateur `node` (uid 1000). Les images
`python:*` (et autres) **ne l'ont pas** → il faut le créer avant la couche sécurité.
Ajoute ceci juste après le `FROM` :

```dockerfile
RUN groupadd --gid 1000 node \
    && useradd --uid 1000 --gid 1000 -m node
```

Puis adapte `postCreateCommand` (ex. `pip install -r requirements.txt`). Le reste
de la couche sécurité est inchangé.

## Rappel sécurité

- Ce template **installe Claude via le réseau au build** (`curl … | bash`). C'est de
  l'outillage de build, pas le runtime de ton app — sans rapport avec une éventuelle
  règle « offline » côté application.
- Les `deny` de `managed-settings.json` sont un garde-fou de dernier recours, pas la
  sécurité principale : c'est l'**isolation du conteneur** qui protège ta machine.
