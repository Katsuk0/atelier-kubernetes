# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** LY William
**Groupe :** I3 APP-RS2

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > la commande affiche 3 noeuds
   > k3d-lab-agent-0, qui sert à exécuter les applications conteneurisées
   > k3d-lab-agent-1, qui sert à exécuter les applications conteneurisées
   > k3d-lab-server-0, qui gère le cluster Kubernetes et coordonne les ressources et décide où les pods doivent être exécutés
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >docker montre les conteneurs en cours d'exécution
   >Un noeud dans k3d est un conteneur docker exécutant k3s qui permet de simuler un cluster de plusieurs noeuds sur une seule machine

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >Les objets suivant ont été crées avec kubectl create deployment:
   >NAME                       READY   STATUS    RESTARTS   AGE
   pod/web-68d995574f-bqjrv   1/1     Running   0          2m3s

   NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
   service/kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP   14m
   service/web          ClusterIP   10.43.173.61   <none>        80/TCP    4m17s

   NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/web   1/1     1            1           4m17s

   NAME                             DESIRED   CURRENT   READY   AGE
   replicaset.apps/web-68d995574f   1         1         1       4m17s

4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Kubernetes a supprimé le pod original et comme deployment souhaite toujours maintenir un pod, le replicaset a automatiquement créé un nouveau pod

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >après la suppresion du pod, le message n'a pas survécu, il avait été ajouté directement dans le système fichiers du conteneur, qui est éphémère et lors de la recréation du pod, kubernetes utilise l'image nginx originale et la page par défaut est restaurée
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >le port forward était connecté au pod supprimé et sa suppresion a entraîné la fermetyure de son espace réseay ce qui a provoqué une erreur de connexion

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > kubectl create sert principalement à créer alors que kubectl apply facilite la gestion et la mise à jour des ressources
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >le selector permet à un service d'identifier les pods auxquels il doit envoyer du trafic en fonction de leurs labels. S'il ne correspond à aucun pod, le service existe toujours mais il ne possède aucun endpoint disponible et le trafic envoyé au service ne peut être transmis à aucun pod

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
````
Les noeuds se trouvent dans k3d-lab-agent-1, k3d-lab-server-0 et k3d-lab-agent-0
NAME                   READY   STATUS              RESTARTS   AGE    IP          NODE               NOMINATED NODE   READINESS GATES
site-75775484b-cjn6q   0/1     ContainerCreating   0          1s     <none>      k3d-lab-agent-1    <none>           <none>
site-75775484b-glszv   0/1     ContainerCreating   0          1s     <none>      k3d-lab-server-0   <none>           <none>
site-75775484b-hdr5b   1/1     Running             0          6m8s   10.42.0.7   k3d-lab-agent-0    <none>           <none>
````


10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > il est passé par plusieurs status:
    > pending, containercreating, errimagepull, imagepullbackoff
    > on regardait avec kubectl get pods -o wide -w
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >le site restait accessible parce qu'il restait au moins un pod fonctionnel et comme le deployment utilise 3 réplicas pendant la mise à jour, kubernetes crée de nouveaux pods avec la nouvelle image avant de supprimer les anciens pods fonctionnels

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://sturdy-space-garbanzo-xjgj46q6jxfp54-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Non, c'est le même type de ressource Kubernetes. La seule différence est que Portainer permet de la créer via une interface graphique, tandis que kubectl utilise la ligne de commande. les deux communiquent avec le même composant kubernetest api server

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** Score : 15 / 18  (l'exercice 3 n'est pas vérifiable automatiquement)
