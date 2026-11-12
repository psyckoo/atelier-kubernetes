# COMPTE RENDU — ATELIER KUBERNETES 2026

**Nom et prénom :** Yannis HAMIDOU  
**Formation :** M2 Informatique

---

# Exercice 1 — Création du cluster

### 1. Combien de nœuds contient le cluster ? Quels sont leurs rôles ?

Le cluster Kubernetes `lab` contient 3 nœuds :

- `k3d-lab-server-0` : nœud serveur (control-plane).
- `k3d-lab-agent-0` : premier agent.
- `k3d-lab-agent-1` : deuxième agent.

Les trois nœuds sont dans l'état `Ready`.

### 2. Quels conteneurs composent le cluster ?

Le cluster a été créé avec k3d, qui utilise Docker pour exécuter les nœuds Kubernetes sous forme de conteneurs.

Les conteneurs principaux sont :

- `k3d-lab-server-0`
- `k3d-lab-agent-0`
- `k3d-lab-agent-1`
- `k3d-lab-serverlb` : load balancer du cluster.

Le port 8081 de la machine est relié au port 80 du load balancer.

---

# Exercice 2 — Premier déploiement

### 3. Quelle est la différence entre un Pod et un Deployment ?

Un Pod est la plus petite unité déployable dans Kubernetes. Il contient un ou plusieurs conteneurs.

Un Deployment permet de gérer les Pods et de maintenir le nombre de réplicas souhaité. Il s'appuie sur un ReplicaSet pour créer, remplacer et mettre à jour les Pods.

### 4. Que se passe-t-il lorsqu'un Pod géré par un Deployment est supprimé ?

Le ReplicaSet détecte que le nombre de Pods disponibles est inférieur au nombre souhaité et crée automatiquement un nouveau Pod pour maintenir le nombre de réplicas défini.

Le Deployment `web` a été créé et son déploiement a été confirmé avec succès.

---

# Exercice 3 — Manipulation des Pods

### 5. Que se passe-t-il lorsqu'on modifie un fichier directement dans un conteneur, puis que le Pod est recréé ?

Une modification effectuée directement dans le système de fichiers du conteneur peut être perdue lorsque le Pod est recréé.

Pour conserver les données, il faut utiliser un volume ou un système de stockage adapté.

### 6. Que se passe-t-il si le Pod ciblé par un port-forward est supprimé ?

Le port-forward associé à ce Pod peut s'interrompre. Il faut alors établir un nouveau port-forward vers le Pod ou le Service approprié.

---

# Exercice 4 — Déploiement déclaratif

### 7. Quelle est la différence entre une commande impérative et un déploiement déclaratif ?

Une commande impérative demande directement à Kubernetes d'effectuer une action.

Un déploiement déclaratif consiste à décrire l'état souhaité dans des fichiers YAML, puis à appliquer ces fichiers avec :

```bash
kubectl apply -f k8s/site/
```

Cette méthode permet de conserver une configuration reproductible et de versionner les manifestes.

### 8. À quoi sert le selector d'un Service ?

Le selector permet au Service de sélectionner les Pods correspondant aux labels définis.

Le Service peut ainsi transmettre le trafic réseau aux Pods concernés.

**Résultats obtenus :**

- Les TODO des manifestes `k8s/site/` sont complétés.
- Le Deployment `site` est disponible.
- Le Deployment utilise la ConfigMap `site-html`.
- Le Service `site` possède des endpoints.

---

# Exercice 5 — Scaling et mises à jour

### 9. Sur quels nœuds les Pods du site sont-ils répartis ?

Les trois Pods du site sont répartis sur les trois nœuds du cluster k3d :

- `k3d-lab-server-0` : 1 Pod.
- `k3d-lab-agent-0` : 1 Pod.
- `k3d-lab-agent-1` : 1 Pod.

Les trois Pods sont en état `Running` et `Ready`.

### 10. Que se passe-t-il lorsqu'on utilise une image inexistante, par exemple nginx:9.99 ?

Lorsqu'une image inexistante est demandée, Kubernetes ne parvient pas à la télécharger.

Le Pod peut alors passer par les états `ErrImagePull` puis `ImagePullBackOff`.

Ce test spécifique n'a pas été confirmé dans les résultats fournis.

### 11. Le site reste-t-il disponible pendant une mise à jour ?

Le Deployment utilise une stratégie de mise à jour progressive (RollingUpdate). Kubernetes remplace progressivement les anciens Pods par les nouveaux.

Une mise à jour de l'image Nginx a été effectuée avec succès. L'historique du Deployment affiche trois révisions :

- Révision 1
- Révision 2
- Révision 3

Le site répond toujours avec le code HTTP `200 OK` après la mise à jour.

---

# Exercice 6 — Portainer et Mario

### 12. Quelle est l'adresse permettant d'accéder à Mario ?

Le jeu Mario a été déployé dans Kubernetes et était accessible depuis le navigateur à l'adresse suivante :

https://silver-system-5gpqj7pqp77f74wv-8585.app.github.dev/

Le Service Kubernetes `mario` expose le port 85 et le port-forward redirige le port local 8585 vers ce Service.

Cette adresse correspond au port-forward initial du cluster Kind. L'accès à Mario depuis le nouveau cluster k3d doit être configuré séparément si nécessaire.

### 13. Comment vérifier que Mario fonctionne et comment Portainer accède-t-il au cluster ?

Pour vérifier le Deployment Mario :

```bash
kubectl get deployment mario
kubectl get pods -l app=mario
```

Le Deployment Mario a été créé dans le cluster k3d et son déploiement a été confirmé avec succès.

Portainer a également été installé dans le namespace `portainer` et son Deployment est disponible.

Portainer et kubectl communiquent avec Kubernetes via l'API Server du cluster auquel ils sont connectés.

---

# Bonus

Les éléments suivants ont été réalisés :

- Création d'un cluster Kubernetes avec k3d.
- Création d'un cluster composé d'un serveur et de deux agents.
- Configuration du port 8081 sur le load balancer.
- Déploiement d'un site Nginx.
- Utilisation d'une ConfigMap pour le contenu HTML.
- Création d'un Service Kubernetes.
- Déploiement d'un premier Deployment `web`.
- Mise à l'échelle du site à trois réplicas.
- Répartition des Pods sur plusieurs nœuds.
- Mise à jour de l'image Nginx.
- Installation de Portainer.
- Déploiement du jeu Mario.
- Installation du contrôleur Ingress Traefik.
- Création de l'Ingress `site`.
- Vérification de l'accès au site avec `curl http://localhost:8081`.
- Ajout d'une `readinessProbe`.
- Définition des limites de ressources du conteneur `site`.

Le test HTTP a renvoyé une réponse `200 OK` et affiché le contenu HTML personnalisé du site.

Les contrôles automatiques du bonus Ingress, de la readinessProbe et des limites de ressources sont tous validés.

---

# Résultat final du script de vérification

Commande exécutée :

```bash
./scripts/check.sh
```

**Score final : 18 / 18**

Tous les contrôles automatiques sont validés :

- Cluster k3d `lab` présent.
- Trois nœuds Ready.
- Port 8081 relié au load balancer.
- Deployment `web` disponible.
- Service `web` présent.
- Manifestes déclaratifs complétés.
- Deployment `site` disponible.
- ConfigMap `site-html` utilisée.
- Service `site` avec des endpoints.
- Au moins trois réplicas prêts.
- Pods répartis sur au moins deux nœuds.
- Image Nginx versionnée.
- Au moins une mise à jour déployée.
- Portainer disponible.
- Mario disponible.
- Ingress fonctionnel sur le port 8081.
- ReadinessProbe configurée.
- Limites de ressources configurées.

**Résultat : 18 / 18 — 100 % de réussite.**

L'exercice 3 n'est pas vérifiable automatiquement par le script.