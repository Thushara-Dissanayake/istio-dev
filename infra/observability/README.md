# Kiali Installation

  https://kiali.io/docs/installation/installation-guide/install-with-helm/

  1. Install Kiali Operator with Kiali CR
  
    a. helm repo add kiali https://kiali.org/helm-charts && helm repo update kiali

    b.  helm install \
          --set cr.create=true \
          --set cr.namespace=istio-system \
          --namespace kiali-operator \
          --create-namespace \
          kiali-operator \
          kiali/kiali-operator

    * If you ever want to uninstall the Kiali Operator, remember to delete the Kiali CR first before uninstalling the operator

  2. Set Prometheus & Grafana and login session expiration time in 'Kiali' CR

      k edit Kiali -n istio-system

        spec:
          deployment:
            accessible_namespaces:
            - '**'
          external_services:
            prometheus:
              url: http://kube-prometheus-stack-prometheus.observability:9090
          login_token:
            expiration_seconds: 86400

      * You may need to restart Kiali operator

      Kiali CR API spec: https://github.com/kiali/kiali-operator/blob/master/crd-docs/cr/kiali.io_v1alpha1_kiali.yaml

   
    k create token kiali-service-account -n istio-system

    istioctl dashboard kiali or k port-forward -n istio-system svc/kiali 20001:20001

---------------------------------------------------------------------------------------------

# kube-prometheus-stack Installation

  kube-prometheus-stack contains Prometheus Operator + Prometheus + Grafana
  https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md
  

  1. helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update prometheus-community

  2. helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --version "48.6.0" --create-namespace --namespace observability 

     helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack --version "52.1.0" --namespace observability

      https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml

  3. Create PodMonitor or ServiceMonitor to expose metrics to Prometheus 
      https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/getting-started.md

      k apply -f podmonitor.yaml -n observability


    k port-forward -n observability svc/kube-prometheus-stack-prometheus 9090:9090
    Test PromQL query: istio_requests_total{reporter="destination",destination_app="rollouts-service"}

    k port-forward -n observability svc/kube-prometheus-stack-grafana 3000:80

---------------------------------------------------------------------------------------------

 Dashboards:
 https://github.com/istio/istio/tree/master/manifests/addons/dashboards


 