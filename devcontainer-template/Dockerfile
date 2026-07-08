# ════════════════════════════════════════════════════════════════════════
#  TEMPLATE DEVCONTAINER SÉCURISÉ — voir README.md
#  Deux couches : STACK (à adapter par projet) puis SÉCURITÉ (identique partout).
# ════════════════════════════════════════════════════════════════════════

# ┌─ COUCHE STACK ─────────────────────────────────────────────────────────┐
# │ SEULE partie à changer selon ton projet.                               │
# │   Node   : node:20-bookworm-slim   (fige la version si besoin : 20.17) │
# │   Python : python:3.12-slim-bookworm  (+ créer l'user 'node', cf README)│
# └────────────────────────────────────────────────────────────────────────┘
FROM node:20-bookworm-slim

# Outils système propres à ta stack (optionnel). Exemples :
#   RUN corepack enable && corepack prepare pnpm@latest --activate
#   RUN pip install poetry

# ┌─ COUCHE SÉCURITÉ ───────────────────────────────────────────────────────┐
# │ Identique dans tous tes projets. En principe : ne pas toucher.          │
# └─────────────────────────────────────────────────────────────────────────┘

# git + curl + certifs (souvent absents des images "slim").
RUN apt-get update \
    && apt-get install -y --no-install-recommends git ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

# Réglages "managed" = garde-fous durs (rm -rf /, git push -f, git reset --hard…).
# Chemin système root-owned → l'agent ne peut pas les désactiver de lui-même.
RUN mkdir -p /etc/claude-code
COPY claude/managed-settings.json /etc/claude-code/managed-settings.json

# ~/.claude doit appartenir à 'node' : l'installeur Claude y écrit downloads/.
# (Le COPY --chown ne couvre que le fichier → on chown le dossier avant.)
RUN mkdir -p /home/node/.claude && chown -R node:node /home/node/.claude
COPY --chown=node:node claude/user-settings.json /home/node/.claude/settings.json

# Non-root : rien ne tourne en root dans le conteneur.
USER node
WORKDIR /workspaces/app

# Claude Code via l'installeur natif, EN TANT QUE node.
# (Le paquet npm global échoue en build Docker : son postinstall, lancé en root,
#  dépose le binaire natif dans le home de root — introuvable pour l'user node.)
RUN curl -fsSL https://claude.ai/install.sh | bash
ENV PATH="/home/node/.local/bin:${PATH}"
