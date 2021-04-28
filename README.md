# kube-verdaccio

## Reference

Verdaccio
https://verdaccio.org/

## Structure

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

## Compiled files

verdaccio
├── configmap.yaml
├── deployment.yaml
├── ingress.yaml
└── service.yaml

## Issues, challenges and solution

1. bitbucket plugin
Issue: A problem setting up a bitbucket plugin where user details are read directly from bitbucket user list. 
Cause: It seems that this requires 2FA, and could not find a similar use case on webs.
Solution: Used a separate htpasswd file and attached as a configmap. This works as there are less than 10 active users.
Improvement: Look into this again.

2. Error message “warn --- invalid address - http://0.0.0.0:tcp://172.20.241.128:4873, we expect a port (e.g. "4873"), host:port (e.g. "localhost:4873") or full url (e.g. "http://localhost:4873/")” 
Issue: Error message was seen.
Cause: This was caused by the verdaccio docker file already using VERDACCIO_PORT=4873  and our setup was overwriting with VERDACCIO_PORT=tcp://172.20.143.185:4873.
Solution: Renamed app so we have env variables VERDACCIO_APP_PORT=tcp://172.20.229.76:4873
