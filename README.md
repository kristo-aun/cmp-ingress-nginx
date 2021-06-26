# Custom Ingress Chart

This chart is based on the Ingress OTS (off-the-shelf) helm chart, retrieved and pinned to a specific upstream version.

Customization:
- ingress to prioritize incoming X-Forwarded-* headers to the ones set in the [template](https://raw.githubusercontent.com/kubernetes/ingress-nginx/helm-chart-3.34.0/rootfs/etc/nginx/template/nginx.tmpl). This is useful if there is another reverse-proxy in front of the ingress controller 

## Install

Download the Ingress chart from the repository to a local copy

    helm dep update

Deploy

    helm upgrade cmp-ingress-nginx . \
        --namespace=kube-system \
        --install

## Verify

Save the controller pod name to a variable

    POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')

Detect installed version    

    kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version

Should produce:

    -------------------------------------------------------------------------------
    NGINX Ingress controller
    Release:       v0.47.0
    Build:         7201e37633485d1f14dbe9cd7b22dd380df00a07
    Repository:    https://github.com/kubernetes/ingress-nginx
    nginx version: nginx/1.20.1

Verify the customization:

    kubectl exec -it $POD_NAME -- cat /etc/nginx/template/nginx.tmpl | grep "X-Forwarded-Host"

Should produce:

    {{ $proxySetHeader }} X-Forwarded-Host       $http_x_forwarded_host;

You can now proceed to deploy Ingress for your applications.

## Uninstall

    helm delete cmp-ingress-nginx

## Helpers

NGINX Ingress controller can be installed via Helm using the chart from the project repository.

    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm search repo ingress-nginx
    helm install ingress-nginx ingress-nginx/ingress-nginx
