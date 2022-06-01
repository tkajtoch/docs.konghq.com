---
title: How to Install Kong Gateway with Helm
toc: true
content-type: how-to
---

This guide shows you how to deploy {{site.base_gateway}} with Helm on your local Kubernetes environment. These steps are known to work on [Docker Desktop Kubernetes](https://docs.docker.com/desktop/kubernetes/) and [Kind](https://kind.sigs.k8s.io/).
Once deployed, Kong will be locally accessible at `https://kong.127-0-0-1.nip.io`. [nip.io](https://nip.io) is used to automatically resolve this domain to localhost. 

The {{site.base_gateway}} software is governed by the
[Kong Software License Agreement](https://konghq.com/kongsoftwarelicense/).
{{site.ce_product_name}} is licensed under an
[Apache 2.0 license](https://github.com/Kong/kong/blob/master/LICENSE).


{% navtabs %}
{% navtab Docker Desktop Kubernetes %}
## Docker Desktop prerequisites

Docker Desktop Kubernetes, is a tool for running a local Kubernetes cluster in Docker Desktop. In this guide you can deploy a Docker Dekstop Kubernetes cluster and then use Helm to install Kong Enterprise Gateway.
## 

- [`Helm 3`](https://helm.sh/)
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/) v1.19 or later
- A `license.json` enterprise license file from Kong
- [Docker Desktop Kubernetes](https://docs.docker.com/desktop/kubernetes/)

{% endnavtab %}
{% navtab Kind Kubernetes %}

## Kind Kubernetes prerequisites

- [`Helm 3`](https://helm.sh/)
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/) v1.19 or later
- A `license.json` enterprise license file from Kong
- [KinD](https://kind.sigs.k8s.io/)
- Ports 80 and 443 available

## Create Kubernetes Cluster

Kind or Kubernetes-in-Docker, is a tool for running local Kubernetes clusters in Docker Containers. In this guide you can deploy a Kind Kubernetes cluster and then use Helm to install Kong Enterprise Gateway. 
To build a local Kubernetes Cluster you have to create a YAML file that contains details about your cluster. Starting from the cat command, and ending with EOF, highlight and copy this entire text block, then paste it into your terminal.

    cat <<EOF > /tmp/kind-config.yaml && kind create cluster --config /tmp/kind-config.yaml
    apiVersion: kind.x-k8s.io/v1alpha4
    kind: Cluster
    name: kong
    networking:
      apiServerAddress: "0.0.0.0"
      apiServerPort: 16443
    nodes:
      - role: control-plane
        extraPortMappings:
        - listenAddress: "0.0.0.0"
          protocol: TCP
          hostPort: 80
          containerPort: 80
        - listenAddress: "0.0.0.0"
          protocol: TCP
          hostPort: 443
          containerPort: 443
    EOF

Verify that the cluster was installed using `kind get clusters`.  This command returns `kong` if the installation was successful.

{% endnavtab %}
{% endnavtabs %}

## Install Cert Manager

Cert Manager provides automation for generating ssl certificates. This Kong deployment will use Cert Manager to provide several required certs.

Once Docker Desktop Kubernetes is enabled, install dependencies:

1. Add the Jetstack Cert Manager Helm repository:

        helm repo add jetstack https://charts.jetstack.io ; helm repo update

2. Install Cert Manager:

        helm upgrade --install cert-manager jetstack/cert-manager \
            --set installCRDs=true --namespace cert-manager --create-namespace


## Configure Kong Gateway

Configuring Kong Gateway requires a namespace and configuration secrets. Our secrets contain Kong's enterprise license, admin password, session configurations, and Postgres connection details. If you do not have a `license.json` file, please contact your account manager.

2. Create Kong namespace for {{site.base_gateway}}:

        kubectl create ns kong

3. Create Kong Enterprise License secret:

        kubectl create secret generic kong-enterprise-license --from-file=license=license.json -n kong

      >These instructions must be run in the directory that contains your `license.json` file.

4. Create Kong config & credential variables:

        kubectl create secret generic kong-config-secret -n kong \
            --from-literal=kong_admin_password=kong \
            --from-literal=portal_session_conf='{"storage":"kong","secret":"super_secret_salt_string","cookie_name":"portal_session","cookie_samesite":"off","cookie_secure":false}' \
            --from-literal=admin_gui_session_conf='{"storage":"kong","secret":"super_secret_salt_string","cookie_name":"admin_session","cookie_samesite":"off","cookie_secure":false}' \
            --from-literal=pg_host="enterprise-postgresql.kong.svc.cluster.local" \
            --from-literal=pg_port="5432" \
            --from-literal=password=kong \
          | kubectl apply -f -

6. Create a SelfSigned certificate issuer:

       cat <<EOF | kubectl apply -n kong -f -
       apiVersion: cert-manager.io/v1
       kind: Issuer
       metadata:
         name: quickstart-kong-selfsigned-issuer-root
       spec:
         selfSigned: {}
       ---
       apiVersion: cert-manager.io/v1
       kind: Certificate
       metadata:
         name: quickstart-kong-selfsigned-issuer-ca
       spec:
         commonName: quickstart-kong-selfsigned-issuer-ca
         duration: 2160h0m0s
         isCA: true
         issuerRef:
           group: cert-manager.io
           kind: Issuer
           name: quickstart-kong-selfsigned-issuer-root
         privateKey:
           algorithm: ECDSA
           size: 256
         renewBefore: 360h0m0s
         secretName: quickstart-kong-selfsigned-issuer-ca
       ---
       apiVersion: cert-manager.io/v1
       kind: Issuer
       metadata:
         name: quickstart-kong-selfsigned-issuer
       spec:
         ca:
           secretName: quickstart-kong-selfsigned-issuer-ca
       EOF
 
## Deploy Kong Gateway

Kong Gateway locally accessible at `https://kong.127-0-0-1.nip.io`. This guide uses [nip.io](https://nip.io) to automatically resolve this domain to localhost.

{:.important}
> The following 4 steps are temporary development steps and will be removed from the guide.
> These steps are required to access the helm-chart before it is merged into production.
> These steps require the [Github CLI](https://cli.github.com/).

1. `gh repo clone Kong/charts ~/kong-charts-helm-project`
2. `cd ~/kong-charts-helm-project/charts/kong`
3. `gh pr checkout 592`
4. `helm dependencies update`

Once all dependencies are installed and ready, deploy Kong Gateway to your cluster:

1. Add the Kong Helm Repo:

        helm repo add kong https://charts.konghq.com ; helm repo update

2. Install Kong:

        helm upgrade --install quickstart --namespace kong \
          --values ./example-values/quickstart-enterprise-licensed-aio.yaml \
          ./

3. Wait for all pods to be in the `Running` state:

        kubectl get po --namespace kong

4. Once all pods are running, open Kong Manager in your browser at [https://kong.127-0-0-1.nip.io](https://kong.127-0-0-1.nip.io). The [nip.io](https://nip.io) domain is a service that automatically resolves the cluster's domain to localhost.

5. Log in with the Super Admin username and password combination: `kong_admin`:`kong`

    {:.note}
    > In Chrome you may receive a "Your Connection is not Private" warning message.  
    > If there is no "Accept risk and continue" option then type `thisisunsafe` while the in the tab to continue.




You can use the Kong Admin API with Insomnia, HTTPie, or cURL, at [https://kong.127-0-0-1.nip.io/api](https://kong.127-0-0-1.nip.io/api)

{% navtabs codeblock %}
{% navtab cURL %}
```sh
curl --silent --insecure -X GET https://kong.127-0-0-1.nip.io/api -H 'kong-admin-token:kong'
```
{% endnavtab %}
{% navtab HTTPie %}
```sh
http --verify=no get https://kong.127-0-0-1.nip.io/api kong-admin-token:kong
```
{% endnavtab %}
{% endnavtabs %}

## Teardown 

{% navtabs %}
{% navtab Docker Desktop Kubernetes %}

The following steps can be used to uninstall Kong Gateway from your system.

1. Remove Kong

        helm uninstall --namespace kong enterprise

2. Delete Kong secretes 

        kubectl delete secrets -nkong kong-enterprise-license
        kubectl delete secrets -nkong kong-config-secret

3. Remove Kong database PVC

        kubectl delete pvc -nkong data-enterprise-postgresql-0

{% endnavtab %}
{% navtab Kind Kubernetes %}

The following steps can be used to uninstall Kong Gateway from your system.

1. Remove Kong

        helm uninstall --namespace kong enterprise

2. Delete Kong secretes

        kubectl delete secrets -nkong kong-enterprise-license
        kubectl delete secrets -nkong kong-config-secret

3. Remove Kong database PVC

        kubectl delete pvc -nkong data-quickstart-postgresql-0

4. Remove Kong Helm Chart Repository
  
        helm repo remove kong
5. Remove cert-manager
  
        helm uninstall --namespace kong cert-manager

6. Remove jetstack cert-manager Helm Repository

        helm repo remove jetstack

7. Destroy your Kind Cluster
  
        kind delete cluster --name=kong
        rm /tmp/kind-config.yaml 



{% endnavtab %}
{% endnavtabs %}


## Conclusion

See the [Kong Ingress Controller docs](/kubernetes-ingress-controller/) for  how-to guides, reference guides, and more.