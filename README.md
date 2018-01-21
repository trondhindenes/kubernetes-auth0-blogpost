# Kubernetes-Auth0
This repo contains the things needed to set up Auth0 as the authentication/authorization backend for Kubernetes, using OpenID Connect.

After following this repo you should be able to:
- Use the Kubernetes Dashboard with authentication pass-thru

### Known issues
- Proper logout/session handling isn't implemented
- Kubernetes supports refresh tokens, tho I haven't gotten that to work yet

### Requirements
- You need an active auth0 subscription, a free one is more than enough. You also need a user in auth0, for example by activating the Google integration, and using your gmail user. Or something.
- You need to have minikube (https://github.com/kubernetes/minikube) up and running locally 

### Do the things
[1. Configure Auth0](docs/01-auth0.md)   
[2. Configure the Kubernetes(minikube) cluster](docs/02-kubernetes.md)   
[3. Deploy `kubernetes-dashboard` and the `mod_oidc` proxy](docs/03-dashboard.md)   

