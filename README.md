# projet-CICD

Pour ArgoCD :<br><br>
kubectl create namespace argocd<br>
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml<br>
kubectl patch svc argocd-server -n argocd --type='json' -p '[{"op": "replace", "path": "/spec/type", "value": "NodePort"}]'<br>
kubectl get all --namespace argocd<br>
kubectl port-forward svc/argocd-server -n argocd 8080:443<br>
On ouvre un autre terminal<br>
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"<br>
On utilise un décodeur de base64 pour le mot de passe<br>
On se connecte à https://localhost:8080<br>
On va dans les settings pour ajouter un repos git<br>
Une fois fait et que le repos est considéré "succesful" on peut aller créer une app argoCD<br>
On va dans le menu "Application" et on ajoute son app avec self-heal et replace d'activé, puis on ajoute le repos git, le chemin des manifests yml et le kubernetes par défaut et le nom du namespace sur lequel on veut que tous soit run, j'ai choisi : kubectl create namespace projet-cicd<br>
