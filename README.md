# TP 5.2 — StatefulSet Kubernetes

## 1. Objectif du TP

Ce TP permet de comprendre la gestion des applications stateful avec Kubernetes, en utilisant un StatefulSet, un Headless Service et un PersistentVolumeClaim (PVC).

## 2. Environnement

- Cluster Kubernetes avec kubeadm
- Namespace : `tp-5-2`
- StatefulSet : `db`
- Pod : `db-0`
- Headless Service : `db`
- Port : `5432/TCP`
- Stockage persistant : `1Gi`
- StorageClass : `local-path`

## 3. Étape 1 — Vérification du cluster

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

Vérifier que les nœuds Kubernetes sont opérationnels et que les composants DNS sont en cours d'exécution.

## 4. Étape 2 — Namespace

```bash
kubectl get namespace tp-5-2
```

Le namespace `tp-5-2` permet d'isoler les ressources du TP.

## 5. Étape 3 — Configuration de l'application

Le fichier `app-config.yaml` contient la configuration de l'application.

```bash
cat app-config.yaml
```

## 6. Étape 4 — StatefulSet et stockage persistant

Le fichier `db.yaml` définit les ressources Kubernetes nécessaires.

```bash
kubectl apply -f app-config.yaml
kubectl apply -f db.yaml
```

Vérifier les ressources :

```bash
kubectl get statefulsets,svc,pvc -n tp-5-2
kubectl get pods -n tp-5-2 -o wide
```

Résultats obtenus :

- StatefulSet `db` : `1/1 READY`
- Pod `db-0` : `Running`
- PVC `data-db-0` : `Bound`
- Capacité du volume : `1Gi`

## 7. Étape 5 — Vérification du DNS Kubernetes

Vérifier les Pods CoreDNS :

```bash
kubectl get pods -n kube-system -o wide | grep coredns
```

Vérifier le Service DNS :

```bash
kubectl get svc kube-dns -n kube-system
```

Créer un Pod de test DNS :

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  -n tp-5-2 \
  --command -- sleep 3600
```

Tester la résolution DNS :

```bash
kubectl exec -n tp-5-2 dns-test -- \
  nslookup kubernetes.default.svc.cluster.local
```

Résultat obtenu : `kubernetes.default.svc.cluster.local` est résolu vers `10.96.0.1`.

Tester la résolution DNS du StatefulSet :

```bash
kubectl exec -n tp-5-2 dns-test -- \
  nslookup db.tp-5-2.svc.cluster.local
```

Résultat obtenu : le nom du Service est résolu vers l'adresse IP du Pod `db-0`.

## 8. Étape 6 — Test de connectivité

Tester l'accès au port de la base de données :

```bash
kubectl exec -n tp-5-2 dns-test -- \
  nc -zv db.tp-5-2.svc.cluster.local 5432
```

Résultat obtenu :

```text
db.tp-5-2.svc.cluster.local (192.168.190.122:5432) open
```

Le test confirme que le port TCP 5432 est accessible depuis le Pod de test.

## 9. Étape 7 — Nettoyage du Pod de test

```bash
kubectl delete pod dns-test -n tp-5-2
```

Cette commande supprime uniquement le Pod de test et conserve le StatefulSet et son stockage.

## 10. Conclusion

Les vérifications effectuées montrent que le StatefulSet `db` est opérationnel, que le PVC est lié, que la résolution DNS fonctionne et que le port TCP 5432 est accessible.

Ces tests valident les fonctionnalités vérifiées pendant ce TP. Ils ne remplacent pas un test d'authentification ou une opération réelle sur la base de données.

## 11. Fichiers du projet

- `app-config.yaml` : configuration de l'application.
- `db.yaml` : définition des ressources Kubernetes.
- `README.md` : documentation des étapes et des tests.

