# Déployer une application sur Kubernetes

## Prérequis

- Avoir un cluster Kubernetes fonctionnel
- Avoir installé `kubectl` sur votre machine
- Avoir installé `helm` sur votre machine

## Déploiement de l'application avec un simple fichier YAML

Créez un fichier `deployment.yaml` avec le contenu suivant :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
    replicas: 3
    selector:
        matchLabels:
            app: my-app
    template:
        metadata:
            labels:
                app: my-app
        spec:
            containers:
              - name: my-app
                image: nginx:latest
                ports:
                - containerPort: 80
```

Appliquez le fichier YAML sur votre cluster Kubernetes :

```bash
kubectl apply -f deployment.yaml
```

Validez que le déploiement a bien été créé :

```bash
kubectl get deployments
```

Validez que les pods ont bien été créés :

```bash
kubectl get pods
```

## Exposer le déploiement

### Exposer le déploiement avec un service de type `LoadBalancer`

Créez un fichier `service.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
    selector:
        app: my-app
    ports:
    - protocol: TCP
      port: 8090
      targetPort: 80
    type: LoadBalancer
```

Appliquez le fichier YAML sur votre cluster Kubernetes :

```bash
kubectl apply -f service.yaml
```

Naviguez vers l'adresse IP du serveur avec 8090 pour accéder à l'application.

### Exposer le déploiement avec un ingress

Créer un fichier `ingress.yaml` avec le contenu suivant :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
spec:
    rules:
    - host: my-app.local
      http:
        paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: my-app
               port:
                 number: 8090
```

Appliquez le fichier YAML sur votre cluster Kubernetes :

```bash
kubectl apply -f ingress.yaml
```

Testez l'accès à l'application à l'aide de la commande `curl` :

```bash
curl -H "Host: my-app.local" http://<IP-ADDRESS>
```

## Mettre à l'échelle le déploiement

Pour mettre à l'échelle le déploiement à 5 répliques :

```bash
kubectl scale deployment my-app --replicas=5
```

Validez que le déploiement a bien été mis à l'échelle :

```bash
kubectl get deployments
```

## Supprimer le déploiement

Pour supprimer le déploiement :

```bash
kubectl delete deployment my-app
kubectl delete service my-app
kubectl delete ingress my-app
```

## Déploiement de l'application avec ArgoCD

### Installation d'ArgoCD

Créez un namespace `argocd` :

```bash
kubectl create namespace argocd
```

Installez ArgoCD :

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Accès à l'interface web

Créez un port-forward pour accéder à l'interface web :

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Accédez à l'interface web à l'adresse `https://localhost:8080` avec le login `admin` et le mot de passe est dans la commande suivante :

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

### Préparez le déploiement dans le repository Git

Clonez le repository Git :

```bash
git clone git@github.com:itta-kubernetes-2024/argocd.git
```

Créez un dossier `my-app-<votre-nom>` :

```bash
cd argocd
mkdir my-app-<votre-nom>
cd my-app-<votre-nom>
```

Créez un fichier `deployment.yaml` avec le contenu suivant :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
    replicas: 3
    selector:
       matchLabels:
          app: my-app
    template:
        metadata:
            labels:
                app: my-app
        spec:
            containers:
            - name: my-app
              image: nginx:latest
              ports:
                - containerPort: 80
```

Créez un fichier `service.yaml` avec le contenu suivant :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
    selector:
        app: my-app
    ports:
    - protocol: TCP
      port: 8099
      targetPort: 80
    type: LoadBalancer
```

Ajoutez les fichiers dans le repository Git :

```bash
git add deployment.yaml service.yaml
git commit -m "Add deployment and service"
git push
```

### Déploiement de l'application

Dans l'interface web d'ArgoCD, cliquez sur `New App` et remplissez les champs suivants :

- Application Name: `my-app`
- Project: `default`
- Namespace: `default`
- Sync Policy: `Automatic`
- Repository URL: `https://github.com/itta-kubernetes-2024/argocd.git`
- Path: `my-app-<votre-nom>`
- Cluster: `in-cluster`

Cliquez sur `Create`.

Validez que l'application a bien été créée :

```bash
kubectl get applications -n argocd
```

Connectez-vous à l'interface web d'ArgoCD et vérifiez que l'application a bien été déployée.
