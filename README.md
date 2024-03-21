# projet-CICD

## Lancement en local de l'application

### Lancer un conteneur MongoDB

```bash
docker run -d -p 27017:27017 --name mongo mongo:latest
```

Cette commande crée un conteneur, une image et un volume pour MongoDB avec la dernière version.

### Lancer le service order

```bash
cd order-service
npm install
npm start
```

### Lancer le service user

```bash
cd user-service
pip install -r requirements.txt
python app.py
```

### Lancer le service frontend

```bash
cd frontend
npm install
npm start
```

## Dockerization

Exécuter cette commande pour lancer l'application
```bash
docker compose up
```

### Push des images dans un repos Docker Hub
```bash
docker tag <IMAGE_ID> necati44/projet-cicd-user:latest
docker push necati44/projet-cicd-user:latest

docker tag <IMAGE_ID> necati44/projet-cicd-order:latest
docker push necati44/projet-cicd-order:latest

docker tag <IMAGE_ID> necati44/projet-cicd-front-end-react:latest
docker push necati44/projet-cicd-front-end-react:latest
```

## ArgoCD

### Configuration initiale

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd --type='json' -p '[{"op": "replace", "path": "/spec/type", "value": "NodePort"}]'
kubectl get all --namespace argocd
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Accéder à l'interface utilisateur d'ArgoCD

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
```

Connectez-vous à https://localhost:8080 avec le mot de passe obtenu ci-dessus.

### Ajouter un dépôt Git

Dans les paramètres, ajoutez un dépôt Git.

### Créer une application ArgoCD

Dans le menu "Application", ajoutez une nouvelle application avec les options "self-heal" et "replace" activées. Spécifiez le dépôt Git, le chemin des manifests YAML, le cluster Kubernetes par défaut et le namespace où l'application sera déployée. J'ai personnellement créé un namespace nommé projet-cicd.
