FROM node:18-slim

# Définir le répertoire de travail dans le conteneur
WORKDIR /local/usr/app

# Copier les fichiers package*.json
COPY package*.json ./

# Installer les dépendances du service
RUN npm install

# Copier les fichiers du service dans le conteneur
COPY . .

# Compilation du service
RUN npm run build

# Exposer le port sur lequel le service écoute
EXPOSE 3000

# Commande pour démarrer le service
CMD ["npm", "start"]