# Kubernetes setup

Disclaimer: I wrote Kubelini because I wanted a lightweight way of provisioning Kubernetes things. You don't have to use it, this guide should work with minikube and any other kubernetes installer for that matter. All you need is a Kubernetes "flavor" that allows you to send custom attributes to the kube-apiserver. Minikube for instance, lets you do that like this:

```bash
minikube start \
      --extra-config=apiserver.Authorization.Mode=RBAC \
      --extra-config=apiserver.Authentication.OIDC.IssuerURL=https://myauth0domain.auth.com/ \
      --extra-config=apiserver.Authentication.OIDC.UsernameClaim=email \
      --extra-config=apiserver.Authentication.OIDC.ClientID="wakkawakka"
``` 

The flags you need to pass are:
`authorization-mode`: Set this to `RBAC`.
`oidc-issuer-url`: This should be set to the value of the `Domain` in your Auth0 client settings. You MUST add a trailing slash. TRAILING. SLASH.
`oidc-username-claim`: Should be set to `email`
`oidc-groups-claim`: Should be set to `groups`. This is why we create the special rule in step 1, so that we have a `groups` attribute in the jwt data that Kubernetes inspects.
`oidc-client-id`: Should be set to the value of the `Client ID` from your Auth0 client settings.

Make sure you have kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/) available locally and on your path.

When you run minikube start, minikube should automatically configure kubectl to "target" your minikube instance. Double-check by running `kubectl get nodes`.

At this poing you should have minikube running. The next step is to configure the Kubernetes dashboard. Minikube comes with an "embedded" version of the Kubernetes dashboard, but we'll set up our own auth0-integrated one.