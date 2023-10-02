while true; do curl http://first-app.default:8080/api/devices && echo "" && sleep 1; done


istioctl proxy-status



-------

https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update prometheus-community

helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --version "48.6.0" --namespace observability 

Create PodMonitor or ServiceMonitor
https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/getting-started.md


k port-forward -n observability svc/kube-prometheus-stack-prometheus 9090:9090

k port-forward -n observability svc/kube-prometheus-stack-grafana 3000:80

----------- Kiali --------

https://kiali.io/docs/installation/installation-guide/install-with-helm/

helm repo add kiali https://kiali.org/helm-charts

Kiali Operator (Kiali server is a more simple alternative option):

helm install \
    --set cr.create=true \
    --set cr.namespace=istio-system \
    --namespace kiali-operator \
    --create-namespace \
    kiali-operator \
    kiali/kiali-operator

Set Prometheus and Grafana in 'Kiali' object
https://github.com/kiali/kiali-operator/blob/master/crd-docs/cr/kiali.io_v1alpha1_kiali.yaml

  spec:
    deployment:
      accessible_namespaces:
      - '**'
    external_services:
      prometheus:
        url: http://kube-prometheus-stack-prometheus.observability:9090


k create token kiali-service-account -n istio-system

k port-forward -n istio-system svc/kiali 20001:20001

-----
 Dashboards:
 https://github.com/istio/istio/tree/master/manifests/addons/dashboards




