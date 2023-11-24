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

## Keycloak Configuration

1- Create a new Realm
In Keycloak, create a new Realm by moving the cursor over Master on the top left menu and clicking Add realm.
+ Choose a name
+ Enable ON
+ Click the Create button

After creating the new realm, you will see that the name on the top left menu will change from Master to the name you choose in the realm creation if don not, switch to the realm you just created.

2- Create a new OIDC client. 

Select Clients on the left menu and click the Create button on the right side of the window. Configure with the settings below.
The Client ID is a name that will be used in the NeuVector configuration.
+ Client Protocol = openid-connect
+ Root URL = NeuVector OpenID Connect Redirect URI
+ Access NeuVector UI and go to Settings > OpenID Connect Settings
+ At the top of the page, you will find the OpenID Connect Redirect URI
+ click on the button Copy to Clipboard
+ Go back to the Keycloak webpage and paste it into the Root URL field.
+ Save the configuration

3- Client Settings Tab

After saving the configuration, you will be redirected to the client configuration in the Settings tab. Configure with the settings below.
Access Type = confidential

4- Client Mappers tab

In the new OIDC client, create Mappers to expose the user's fields. Select Mappers tab and configure with the settings below.
+ Click on the Creat button on the right side of the page.
+ Choose a Name for your Mapper.
+ Mapper Type = Group Membership
+ Token Claim Name = groups
+ Full group path = OFF
+ Add to ID token = OFF
+ Add to access token = OFF
+ Add to userinfo = ON
+ Save the configuration

5- Information required to configure NeuVector

Endpoint configuration
+ Select Realm Settings on the left menu.
+ On the Endpoints field, click on OpenID Endpoints Configuration. You will be redirected to another page.
+ Copy the URL in the first line, right after issuer, without quotes.

Client ID
Select Clients on the left menu and take note of the Client ID created. Same from step 2.

+ Secret
+ Select Clients on the left menu and select the Client ID created.
+ Select the Credentials tab.
+ Copy the Secret field.

### Configuring Keycloak in NeuVector

+ OpenID configuration
+ Access the NeuVector UI and select Settings on the left menu.
+ Identity Provider Issuer = Copy the URL from the Keycloak issuer from step 5.
+ Client ID = Copy the Client ID name created in step 2.
+ Client Secret = Copy the Secret collected in step 5.
+ Group Claim = groups
+ Default Role = None
+ Add the groups created inside Keycloak to authorize the users to access the NeuVector UI.
+ Select Enable
+ Submit the configuration

You should see a green pop-up at the NeuVector bottom page showing the message "Server Saved!"
In your next login, you should see a "Login with OpenID" option in the NeuVector UI. Selecting this option will redirect to the Keyclaok webpage to authenticate the user. If the authentication works and the user is part of an authorized group, you will be redirected to the NeuVector UI.

### When does Keycloak require SSL?
By default, Keycloak requires SSL for external requests. This means that HTTPS requires a valid certificate and signed by a trusted Certificate Authority.
You can use Let`s Encrypt, GoDaddy, and many other trusted Certificate Authorities to create, issue, and sign your certificate. Using self-signed certificates or certificates signed by internal Certificate Authorities is currently not supported.
If using self-signed certificates to a certificate signed by a non-trusted CA you should see the following error in the controller pods.

```
|ERRO|CTL|rest.validateOIDCServer: Failed to discover OpenID Connect endpoints - error=Get "https://xxxxx/.well-known/openid-configuration": x509: certificate signed by unknown authority
```
> [!NOTE] 
> Instead of creating a trusted signed certificate, you can disable SSL/HTTPS by following the steps below:

In step #6 - How to Install Keycloak,  it is required to modify the variable KEYCLOAK_FRONTEND_URL and add two other variables to disable HTTPS and enable HTTP.
```
extraEnv: |
  - name: KEYCLOAK_USER
    value: admin
  - name: KEYCLOAK_PASSWORD
    value: admin
 
// Replace https by http. 
  - name: KEYCLOAK_FRONTEND_URL
    value: "http://ralmeida2-keycloak.do.support.rancher.space/auth/"
 
// Add the following variables
  - name: KC_HTTPS_ENABLED
    value: "false"
  - name: KC_HTTP_ENABLED
    value: "true"
```

After updating the Helm values.yaml, execute the command below.
```
helm upgrade --install keycloak codecentric/keycloak --namespace keycloak --values codecentric.yaml
```
