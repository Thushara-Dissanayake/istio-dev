# istio Installation Option 1: Using Helm Charts 

  https://istio.io/v1.16/docs/setup/install/helm/

  1. helm repo add istio https://istio-release.storage.googleapis.com/charts && helm repo update istio

  2. kubectl create namespace istio-system

  3. helm install istio-base istio/base -n istio-system

  4. helm install istiod istio/istiod -n istio-system --wait

  5. Enable istio. Two options:
    a. Enable at ns level
      k apply -f namespace.yaml

    b. Enable at pod level
      sidecar.istio.io/inject: true

  Uninstall:
    1. helm delete istiod -n istio-system
    2. helm delete istio-base -n istio-system

# istio Installation Option 2: Using istio Operator 

    https://istio.io/latest/docs/setup/install/operator/
    This relieves you of the burden of managing different istioctl versions

    1. Install istio operator
        
        istioctl operator init

    2. Install istio
        k apply -f istioOperator.yaml

    3. Enable sidecar injection
      k apply -f namespace.yaml

#  istio Installation Option 3: Using istioctl

    https://istio.io/latest/docs/setup/install/istioctl/

    istioctl install --set profile=minimal 
    
      istioctl profile dump minimal
      Available Profiles: https://istio.io/latest/docs/setup/additional-setup/config-profiles/

---------------------------------------------------------------------------------------------

istio uninstallation: 
  istioctl uninstall --purge

---------------------------------------------------------------------------------------------

istioctl proxy-status

istioctl dashboard envoy canary-rollout-5c8b876658-lwfmg

k get mutatingwebhookconfigurations.admissionregistration.k8s.io

istioctl proxy-config all <pod>
istioctl proxy-config all <pod> -o json

istioctl proxy-config cluster <pod>








