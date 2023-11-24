# keycloak

### Keycloak Helm Chart
https://github.com/codecentric/helm-charts/tree/master/charts/keycloak

### Keycloak Repository
https://quay.io/repository/keycloak/keycloak?tab=tags&tag=latest

### Good video explaining how to install Keycloak
https://www.youtube.com/watch?v=1feLDnEmsdE


### Keycloak Installation
1 - Add KeyCloak Repo

```yaml
helm repo add codecentric https://codecentric.github.io/helm-charts
```

2 - Create PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-keycloak-postgresql-0
spec:
  storageClassName: ""
  claimRef:
    name: data-keycloak-postgresql-0
    namespace: keycloak
  capacity:
   storage: 8Gi
  accessModes:
   - ReadWriteOnce
  hostPath:
    path: "/mnt/"
```

3 - Helm show values

```yaml
helm show values codecentric/keycloak --version=17.0.3 > ~/Documents/keycloak-test/codecentric.yaml
```

4 - Edit Helm Chart

```yaml
n// change the namespace
 
namespace: keycloak
 
---
// define a tag, which is the desired Keycloak version
image:
  # The Keycloak image repository
  repository: docker.io/jboss/keycloak
  # Overrides the Keycloak image tag whose default is the chart appVersion
  tag: "16.1.1"  
 
---
// Change the service type to NodePort
service:
  type: NodePort
 
---
// Add the following variables
extraEnv: |
  - name: KEYCLOAK_USER
    value: admin
  - name: KEYCLOAK_PASSWORD
    value: admin
  - name: KEYCLOAK_FRONTEND_URL
    value: "https://ralmeida-keycloak.do.support.rancher.space/auth/"
 
---
// Add volume permission to keycloak
volumePermissions:
  enabled: true
 
---
// Add volume permission to Postgresql
postgresql:
  volumePermissions:
     enabled: true
```

5 - Install keycloak

```yaml
helm upgrade --install keycloak codecentric/keycloak --set volumePermissions.enabled=true --set postgresql.volumePermissions.enabled=true --values codecentric.yaml
```
After installation is completed verify if pods are running and the PV and PVC status
```
╰$ k -n keycloak get pods
NAME                    READY   STATUS    RESTARTS   AGE
keycloak-0              1/1     Running   0          14m
keycloak-postgresql-0   1/1     Running   0          23m
 
 
╰$ k get pv             
NAME                         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS   REASON   AGE
data-keycloak-postgresql-0   8Gi        RWO            Retain           Bound    keycloak/data-keycloak-postgresql-0                           40m
 
 
╰$ k -n keycloak get pvc
NAME                         STATUS   VOLUME                       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-keycloak-postgresql-0   Bound    data-keycloak-postgresql-0   8Gi        RWO                           21m
```

obs: to uninstall keycloak

```yaml
helm uninstall keycloak
```
