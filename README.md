# kube-verdaccio

---
## Reference

Verdaccio
https://verdaccio.org/


### User creation steps

1. Create a user with htpasswd (e.g.) testuser
```
htpasswd -n testuser
New password:
Re-type new password:
testuser:$apr1$dMWhPQ1T$mpc0oMK1sSXBmZb88yz551
```
2. Add `testuser:$apr1$dMWhPQ1T$mpc0oMK1sSXBmZb88yz551` in htpasswd file


---
## AKS Usage

### Reference
1. https://azure.microsoft.com/en-us/free/
2. https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal
3. https://docs.microsoft.com/en-us/azure/aks/ingress-static-ip

### Setting up
**1. Create your Azure free account**

**2. Follow the document 2 above**
- Create AKS cluster

**3. Follow the document 3 above**
- Create an ingress controller
- Install cert-manager
- Create a CA cluster issuer

**4. Deploy kube-verdaccio**
- Update the hosts and host to the DNS name you created with
```
kubectl apply -f kube-verdaccio
```

**5. Follow the document 2 for the cleanup steps**


### Structure

```
/home/<username>
├── certificates.yaml - staging
├── certificates-prod.yaml - prod
├── cluster-issuer.yaml - staging
└── cluster-issuer-prod.yaml - prod

/home/<username>/kube-verdaccio
├── README.md
├── configmap.yaml
├── deployment.yaml
├── ingress.yaml
└── service.yaml
```

### Issues and workaround:
**1. Error: ValidationError(Certificate.spec): unknown field "acme" in**
```
error validating "certificates-prod.yaml": error validating data: ValidationError(Certificate.spec): unknown field "acme" in io.cert-manager.v1.Certificate.spec; if you choose to ignore these errors, turn validation off with --validate=false

error validating "certificates.yaml": error validating data: ValidationError(Certificate.spec): unknown field "acme" in io.cert-manager.v1alpha2.Certificate.spec; if you choose to ignore these errors, turn validation off with --validate=false
```
**Workaround**
https://github.com/MicrosoftDocs/azure-docs/issues/52418

**2. Error: Warning: deprecate: multiple logger configuration is deprecated**
```
 warn --- config file  - /verdaccio/conf/config.yaml
(node:8) Warning: deprecate: multiple logger configuration is deprecated, please check the migration guide.
```
This is because of Verdaccio 5.X image explained here -
https://verdaccio.org/blog/2021/04/14/verdaccio-5-migration-guide#pretty-loggin

**Workaround**
Use version 4.X or follow above guide.


---
## EKS Usage

### Structure

```
verdaccio
├── config.yaml.j2
├── configmap.jsonnet
├── container
│   └── Dockerfile --- custom Dockerfile uploaded to ECR
├── deployment.jsonnet
├── htpasswd.j2 --- keeps password details
├── ingress.jsonnet
├── main.jsonnet
└── service.jsonnet
```

### Compiled files

```
verdaccio
├── configmap.yaml
├── deployment.yaml
├── ingress.yaml
└── service.yaml
```

### Issues, challenges and solution

1. bitbucket plugin
- Issue: A problem setting up a bitbucket plugin where user details are read directly from bitbucket user list. 
- Cause: It seems that this requires 2FA, and could not find a similar use case on webs.
- Solution: Used a separate htpasswd file and attached as a configmap. This works as there are less than 10 active users.
- Improvement: Look into this again.

2. Error message “warn --- invalid address - http://0.0.0.0:tcp://172.20.241.128:4873, we expect a port (e.g. "4873"), host:port (e.g. "localhost:4873") or full url (e.g. "http://localhost:4873/")” 
- Issue: Error message was seen.
- Cause: This was caused by the verdaccio docker file already using VERDACCIO_PORT=4873  and our setup was overwriting with VERDACCIO_PORT=tcp://172.20.143.185:4873.
- Solution: Renamed app so we have env variables VERDACCIO_APP_PORT=tcp://172.20.229.76:4873
