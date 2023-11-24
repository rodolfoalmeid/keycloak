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
   -ReadWriteOnce
  local:
   path: /mnt/

volumePermissions:
  enabled: true

postgresql:
  volumePermissions:
     enabled: true
```

3 - Helm show values

```yaml
helm show values codecentric/keycloak --version=17.0.3 > ~/Documents/keycloak-test/codecentric.yaml
```

4 - Edit Helm Chart

```yaml
namespace: keycloak

---
tag: "16.1.1"

---
service:
  # Annotations for headless and HTTP Services
  annotations: {}
  # Additional labels for headless and HTTP Services
  labels: {}
  # key: value
  # The Service type
  type: NodePort

---
extraEnv: |
  - name: KEYCLOAK_USER
    value: admin
  - name: KEYCLOAK_PASSWORD
    value: admin
  - name: KEYCLOAK_FRONTEND_URL
    value: "https://ralmeida-keycloak.do.support.rancher.space/auth/"
```

5 - Install keycloak

```yaml
helm upgrade --install keycloak codecentric/keycloak --set volumePermissions.enabled=true --set postgresql.volumePermissions.enabled=true --values codecentric.yaml
```

obs: to uninstall keycloak
