#### Changes:

- Cleared k8s/dnsadmin-service-account.json
- Removed postgres IP from k8s/production.config.yml
- Removed cluster details & email from k8s/cert-manager.yml (ClusterIssuer) and reverted to staging
- Removed web/docamole.key and web/docamole.crt

## Setup

To install tools and setup permissions for gke, run the datawire/blackbird installer:

Setup steps will assume you have kubectx/kubens installed on the command-line and the proper gke context/permissions

    curl -L https://d6e.co/2EAPL1u | bash

## Deploy

Deploy cert-manager

There seems to be an issue where the cloud provider secret doesn't get created properly, so we'll do that manually

    kubectl apply -f k8s/cert-manager.yml
    kubens cert-manager
    kubectl create secret generic clouddns --from-file=k8s/dnsadmin-service-account.json

Setup the docamole project/namespace, create configmaps, secrets, and a certificate:

    kubectl apply -f k8s/production.config.yml
    kubens docamole
    kubectl apply -f k8s/production.secrets.yml
    kubectl apply -f k8s/production.cert.yml

**Notice**
At this point you need to wait for the certificate to be issued, it can take some time for letsencrypt to verify your domain, so go grab yourself a cup of coffee :coffee:

Before continuing, verify that the certificate has been issued:

    kubectl describe certificate docamole-com
    kubectl describe secret ambassador-certs

You should see an event stating that the certificate was issued successfully, and the secret should have tls.crt and tls.key with data

Now we're ready to deploy!

    forge deploy

At this point it's just a matter of waiting for the loadbalancer to be assigned a public IP, and then updating the A records.
