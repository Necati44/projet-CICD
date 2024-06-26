# projet-CICD
Travail réalisé par Syméon ROUGEVIN-BÂVILLE et Necati ÜNLÜ
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

Exécutez cette commande pour lancer l'application
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

## Pipeline CI

Deux workflows sont configurés, le premier, "Build and Push Docker Images", qui permet le build et push des images dockers sur un repos Docker Hub avec comme tag latest et le sha du github, déclenché à partir d'un push sur n'importe quelle branche. Et le second, "Versioning Pipeline", qui est enclenché uniquement lors du push de branche versioning avec comme tag, le nom de la branche. Exemple: 1.0.0

## Kubernetes

Les manifests pour Kubernetes se trouvent sur un autre repos : https://github.com/Necati44/microapp-deploy
<br>
Les fichiers de configuration .yaml pour Kubernetes ont été créé dans le dossier **manifests**. Dans install.yaml se trouve toutes les nodes nécessaires à lancer l'application. Et dans le dossier base on peut trouver chacun de ces nodes dans son propre fichier.
<br>
<br>

Pour exécuter la configuration yaml
```bash
kubectl create -f https://raw.githubusercontent.com/Necati44/microapp-deploy/main/manifests/install.yaml
```

Pour supprimer la configuration yaml
```bash
kubectl delete -f https://raw.githubusercontent.com/Necati44/projet-CICD-kubernetes/main/manifests/install.yaml
```

**Bonus:** Créez un secret contenant l'username et le password du service bd :
```bash
kubectl create secret generic db-credentials --from-literal=username=root --from-literal=password=example --namespace=projet-cicd
```
Pour l'instant j'ai mis dans le install.yaml le Secret avec le username et password en dur donc pas sécurisé, il vaut mieux utiliser cette commande même si ce n'est pas automatisé.
<br>
<br>
Pour accéder au service front-end-react il faut faire un port-forward
```bash
minikube service -n projet-cicd front-end-react
```

Et il faut ensuite en faire de même pour que le front puisse communiquer avec les autres services
```bash
kubectl port-forward svc/db -n projet-cicd 27017:27017
kubectl port-forward svc/user -n projet-cicd 3001:3001
kubectl port-forward svc/order -n projet-cicd 3002:3002
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
*ou utiliser un décodeur en ligne: https://www.base64decode.org/*.
<br>
<br>
Connectez-vous à https://localhost:8080 avec le mot de passe obtenu ci-dessus.

### Ajouter un dépôt Git

Dans les paramètres, ajoutez ce dépôt Git. HTTPS fonctionne bien.
![image](https://github.com/Necati44/projet-CICD/assets/78152671/a12db494-6508-4dd8-b42e-a18eb2ce2b9b)

Si le status de connexion est "Successful", cela signifie que la manipulation a été réussi.
![image](https://github.com/Necati44/projet-CICD/assets/78152671/1831b9cd-ef0c-4423-af1c-b3dceec614bc)

### Créer une application ArgoCD

Dans le menu "Application", ajoutez une nouvelle application avec les options suivantes: Dans **GENERAL** mettez la **SYNC POLICY** en "Automatic" et cochez "SELF HEAL", puis en dessous de **PRUNE PROPAGATION POLICY** cochez "REPLACE".
<br><br>
Ou plus simplement, cliquez sur **EDIT AS YAML** et copié/collé cette configuration :
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: projet-cicd
spec:
  destination:
    name: ''
    namespace: ''
    server: ''
  source:
    path: ''
    repoURL: ''
    targetRevision: HEAD
  sources: []
  project: default
  syncPolicy:
    automated:
      prune: false
      selfHeal: true
    syncOptions:
      - Replace=true

```
Pour **SOURCE** sélectionnez le dépot Git que vous ajoutez précédemment et en spécifiant le chemin où se trouve les fichiers de configurations Kubernetes, donc **manifests**.
![image](https://github.com/Necati44/projet-CICD/assets/78152671/2eb1d0cd-e7bf-4ff1-9e7c-ad27025a9f6f)

Finalement, **DESTINATION** on choisi le cluster Kubernetes par défaut et le namespace Kubernetes où l'application sera déployée. Et pour le namespace le notre s'appelle projet-cicd.
![image](https://github.com/Necati44/projet-CICD/assets/78152671/ebb5f30e-aead-480f-84a4-624f487d832d)

L'application est maintenant prête et se synchronisera d'elle même avec le repos **[projet-cicd-kubernetes](https://github.com/Necati44/projet-CICD-kubernetes.git)**.
![image](https://github.com/Necati44/projet-CICD/assets/78152671/b7c5ba2f-b4a6-47b6-aa81-9b37da10cc28)

**Bonus:** Créez un secret contenant l'username et le password du service bd :
```bash
kubectl create secret generic db-credentials --from-literal=username=root --from-literal=password=example --namespace=projet-cicd
```
Pour l'instant j'ai mis dans le install.yaml le Secret avec le username et password en dur donc pas sécurisé, il vaut mieux utiliser cette commande même si ce n'est pas automatisé.
<br>
<br>
Pour accéder au service front-end-react il faut faire un port-forward
```bash
kubectl port-forward svc/front-end-react -n projet-cicd 3000:3000
```
ou si le port sur lequel se trouve le front ne vous importe pas
```bash
minikube service -n projet-cicd front-end-react
```

Et il faut ensuite en faire de même pour que le front puisse communiquer avec les autres services
```bash
kubectl port-forward svc/db -n projet-cicd 27017:27017
kubectl port-forward svc/user -n projet-cicd 3001:3001
kubectl port-forward svc/order -n projet-cicd 3002:3002
```
