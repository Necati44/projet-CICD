FROM node:18-slim

# Installer tini pour une meilleure gestion des processus
RUN apt-get update && \
    apt-get install -y --no-install-recommends tini && \
    rm -rf /var/lib/apt/lists/*

# Définir le répertoire de travail dans le conteneur
WORKDIR /usr/local/app

# have nodemon available for local dev use (file watching)
RUN npm install -g nodemon

# Copier les fichiers du service dans le conteneur
COPY package*.json ./


RUN npm ci && \
 npm cache clean --force && \
 mv /usr/local/app/node_modules /node_modules

COPY . .

ENV PORT 3002
EXPOSE 3002

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]