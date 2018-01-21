# Kubernetes-dashboard

In this section we'll get the Kubernetes Dashboard up and running. Kubernetes-Dashboard is a web app that provides a nice and simple gui for interacting with Kubernetes.

Note: There are many ways to configure traffic routing to a Kubernetes pod. In this example we'll keep it simple, and simply map the pod to a port on the minikube cluster. In production you'll likely use a Service and an Ingress to do it.

How it works:
We'll set up an "Authentication Proxy" based on Apache2 and the `mod_oidc` project. When a client requests the dashboard URL, it hits the auth proxy. If the client isn't already authenticated, mod_oidc redirects the client to Auth0 for logon.

The "source code" for the auth0 proxy is included in the `kube_auth0_proxy` folder of this repo, and setup to auto-build on https://hub.docker.com/r/trondhindenes/kubernetes-auth0/ so you don't have to build it yourself. You'll find this image referenced in the deployment spec described in the following sections.

When the client is logged-in to Auth0, mod_oidc is able to pass an `Authorization` header containig the Auth0 `id_token` to the Kubernetes Dashboard. The Kubernetes Dashboard passes the Bearer token in the Authorization header directly to the Kubernetes apiserver it's talking to, so that everything happens in the context of the logged-in user. Kubernetes uses the `Client Id` we already configured to verify the user, either using the user's email or the user's group membership.

It's important to note that the kubernetes apiserver does _not_ verify the jwt with a secret token. It simply assumes that the jwt sent to it comes from a trusted and verified source.

All of this is contained in the folder `dashboard_deployment` in this repo. 
It's worth noting that you'll likely want to change the configuration a bit. You'll probably use either a Load balancer or an Ingress to expose the dashboard. However, I wanted a setup that doesn't make any assumptions, so I'm using a NodePort service type. This means that Kubernetes will assign an arbitrary port to the dashboard. This also means that we'll have to run things in the right order:

Start with running the following command to get the ip address of your minikube instance:
`minikube ip`
We'll map the Kubernetes Dashboard to `http://<minikube ip>:9000`

Edit the `02_deployment` file by replacing `<minikubeip>` with the actual ip address of your minikube instance.
Also in the same file, set the correct values for the following environment variables:
`auth0clientid`   
`auth0clientsecret`   
`auth0domain`   

When this is done, it's time to deploy!
`kubectl apply -f dashboard_deployment` - this will deploy all files in the folder to Kubernetes. The file `03_auth.yml` contains the `ClusterRole`/`ClusterRoleBinding` that allows users with the right group membership to access Kubernetes.

At this point you need to go back to your Auth0 client settings and add the url to the list of allowed callback urls (in addition to the existing `openidconnect.net`. Make sure you add `http://<minikube ip>:9000/`.)


Now if you access the Kubernetes node ip/service port in your browser, you _should_ be taken to an Auth0 login prompt and then redirected to the Kubernetes dashboard, which should look something like this:

![screenshot from 2017-12-25 20-46-15](https://user-images.githubusercontent.com/1747120/34342587-d58f0c98-e9b4-11e7-9570-ba3383361a53.png)


As a last test you can remove the `KubernetesAdmins` group from your Auth0 user, and open up a new browser to perform a "fresh" login session. You should be presented by the "Access Denied" message in the dashboard again.

So that's pretty much it. Kubernetes Dashboard is an awesome tool, especially for people who don't have a lot of Kubernetes expertise, and by following this guide you can provide limited access to your cluster without sharing "root" credentials.

You'll find more information about Kubernetes' OpenID Connect flow here:
https://kubernetes.io/docs/admin/authentication/#openid-connect-tokens