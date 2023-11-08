################### ArgoRollout with GW-API/istio ###################

1. Install Argo Rollout

    helm repo add argo https://argoproj.github.io/argo-helm && helm repo update argo

        - https://github.com/argoproj/argo-helm/blob/main/charts/argo-rollouts/values.yaml

    helm upgrade argo-rollouts argo/argo-rollouts --version 2.32.2 -f values.yaml --create-namespace --namespace argo-rollouts --install

2. Attach IAM policy to EKS node group IAM role to Get metric data from cloudwatch (Optional - If Cloudwatch is used in Analysis template)

3. Set AWS_REGION variable in argorollout deployment  (Optional - If Cloudwatch is used in Analysis template)

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: argo-rollouts
    spec:
    template:
        spec:
        containers:
        - name: argo-rollouts
            env:
            - name: AWS_REGION
                value: eu-central-1

4. k apply -f analysis-template.yaml -n test (optional)
    * update the load balancer value in AnalysisTemplate first

5. Configurations need to be done before Rollout deployment

    a. If you use GW-API to do rollouts,
        https://github.com/argoproj-labs/rollouts-plugin-trafficrouter-gatewayapi

        i. Install GW-API CRDs
            k apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml

        ii. Install GW-API plugin and configure the Argo controller
            k apply -f argo-rollouts-config.yaml

            ** After this, restart rollout controllers
                k delete pod --all -n argo-rollouts

        iii. Create cluster role for GW-API resources
            k apply -f gateway-controller-role.yaml

        iv. Bind it to Argo controller 
            k apply -f rollbinding.yaml


    b. Apply Ingress, API-GW and HTTPRroute 
            k apply -f ingress.yaml
            k apply -f gw-api.yaml
            k apply -f httproute.yaml

    b. Create application services
        k apply -f services.yaml

6. Deploy Rollout 
    k apply -f rollout.yaml
        
----------------------------------------------------

k  argo rollouts get rollout canary-rollout --watch 

k argo rollouts list rollouts

k argo rollouts dashboard

k argo rollouts promote canary-rollout 

k argo rollouts abort canary-rollout

k get analysisrun


Rollout API Specification: https://argoproj.github.io/argo-rollouts/features/specification/