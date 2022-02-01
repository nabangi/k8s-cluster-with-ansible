k8s-istio.md

## First was to create the namespace 

          `kubectl create namespace istio-system`


The intention was to use helm chart to istio but I got time out issues with the repos.

Embarked on downloading istio

```
curl -L https://istio.io/downloadIstio | sh -

cd istio-1.12.0

export PATH=$PWD/bin:$PATH
```

## Install Istio

          `istioctl install`

          `kubectl label namespace default istio-injection=enabled`

Deploy an application in the cluster
Sample microservice app 

          `kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`

## Configure automatic Envoy Proxy Injection


          `kubectl label namespace default istio-injection=enabled`

# challenges I need some more time to resolve, issues to do with the loadbalancer in the cluster

          `istioctl analyze`  
```
Warning [IST0103] (Pod default/nginx-blue-7699fc7b8b-kzjgc) The pod is missing the Istio proxy. This can often be resolved by restarting or redeploying the workload.
Warning [IST0103] (Pod default/nginx-green-698797fb8c-5szl6) The pod is missing the Istio proxy. This can often be resolved by restarting or redeploying the workload.
Warning [IST0103] (Pod default/nginx-grey-7c45bc4899-lcw57) The pod is missing the Istio proxy. This can often be resolved by restarting or redeploying the workload.
```
### Determining the ingress IP and Port

          `kubectl get svc istio-ingressgateway -n istio-system`
```
NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.64.0.252   10.64.0.114   15021:30305/TCP,80:31952/TCP,443:30830/TCP,31400:32693/TCP,15443:31053/TCP   41m
```

## Get kiali Dashboard

```
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system
```
## Access Dashboard

          `istioctl dashboard kiali`


# Blue/Green Deployment model

Working to grasp it well but I already get the drift it brings the aspect of continous deployment and operational excellence.

I have learnt interresting pespectives from this infrastructure setup which is already going to play out in my subsequent deployments going forward

Thank you

          `kubectl get all -n istio-system`
```
NAME                                        READY   STATUS              RESTARTS   AGE
pod/grafana-cf9797cf8-bls45                 0/1     Evicted             0          10m
pod/grafana-cf9797cf8-lbgzj                 0/1     Evicted             0          7m35s
pod/grafana-cf9797cf8-zrsdz                 1/1     Running             0          69s
pod/istio-egressgateway-555fdf6757-5jfck    0/1     Running             0          36m
pod/istio-ingressgateway-7665b8458c-jq7rw   0/1     Running             0          36m
pod/istiod-64d957d6bc-kpstn                 1/1     Running             0          36m
pod/jaeger-5f65fdbf9b-s97ht                 1/1     Running             0          10m
pod/kiali-79866d6f79-p4zj6                  1/1     Running             0          10m
pod/prometheus-8945b4d5-brcr6               0/2     ContainerCreating   0          10m

NAME                           TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                                                                      AGE
service/grafana                ClusterIP      10.64.0.242   <none>        3000/TCP                                                                     10m
service/istio-egressgateway    ClusterIP      10.64.0.159   <none>        80/TCP,443/TCP                                                               36m
service/istio-ingressgateway   LoadBalancer   10.64.0.25    10.64.0.114   15021:31586/TCP,80:32081/TCP,443:31038/TCP,31400:32489/TCP,15443:31318/TCP   36m
service/istiod                 ClusterIP      10.64.0.130   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        36m
service/jaeger-collector       ClusterIP      10.64.0.46    <none>        14268/TCP,14250/TCP,9411/TCP                                                 10m
service/kiali                  ClusterIP      10.64.0.29    <none>        20001/TCP,9090/TCP                                                           10m
service/prometheus             ClusterIP      10.64.0.129   <none>        9090/TCP                                                                     10m
service/tracing                ClusterIP      10.64.0.3     <none>        80/TCP,16685/TCP                                                             10m
service/zipkin                 ClusterIP      10.64.0.117   <none>        9411/TCP                                                                     10m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                1/1     1            1           10m
deployment.apps/istio-egressgateway    0/1     1            0           36m
deployment.apps/istio-ingressgateway   0/1     1            0           36m
deployment.apps/istiod                 1/1     1            1           36m
deployment.apps/jaeger                 1/1     1            1           10m
deployment.apps/kiali                  1/1     1            1           10m
deployment.apps/prometheus             0/1     1            0           10m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-cf9797cf8                 1         1         1       10m
replicaset.apps/istio-egressgateway-555fdf6757    1         1         0       36m
replicaset.apps/istio-ingressgateway-7665b8458c   1         1         0       36m
replicaset.apps/istiod-64d957d6bc                 1         1         1       36m
replicaset.apps/jaeger-5f65fdbf9b                 1         1         1       10m
replicaset.apps/kiali-79866d6f79                  1         1         1       10m
replicaset.apps/prometheus-8945b4d5               1         1         0       10m
```
## Istio proxy errors and troubleshooting

          `kubectl get mutatingwebhookconfiguration istio-sidecar-injector -o yaml | grep "namespaceSelector:" -A5`
```
          f:namespaceSelector:
            f:matchExpressions: {}
          f:objectSelector:
            f:matchExpressions: {}
          f:rules: {}
          f:sideEffects: {}
--
          f:namespaceSelector:
            f:matchExpressions: {}
          f:objectSelector:
            f:matchExpressions: {}
          f:rules: {}
          f:sideEffects: {}
--
          f:namespaceSelector:
            f:matchExpressions: {}
          f:objectSelector:
            f:matchExpressions: {}
          f:rules: {}
          f:sideEffects: {}
--
          f:namespaceSelector:
            f:matchExpressions: {}
          f:objectSelector:
            f:matchExpressions: {}
          f:rules: {}
          f:sideEffects: {}
--
          f:namespaceSelector:
            f:matchLabels:
              .: {}
              f:istio.io/deactivated: {}
          f:objectSelector:
            f:matchLabels:
--
          f:namespaceSelector:
            f:matchLabels:
              .: {}
              f:istio.io/deactivated: {}
          f:objectSelector:
            f:matchLabels:
--
          f:namespaceSelector:
            f:matchLabels:
              .: {}
              f:istio.io/deactivated: {}
          f:objectSelector:
            f:matchLabels:
--
          f:namespaceSelector:
            f:matchLabels:
              .: {}
              f:istio.io/deactivated: {}
          f:objectSelector:
            f:matchLabels:
--
  namespaceSelector:
    matchExpressions:
    - key: istio.io/rev
      operator: In
      values:
      - default
--
  namespaceSelector:
    matchExpressions:
    - key: istio.io/rev
      operator: DoesNotExist
    - key: istio-injection
      operator: DoesNotExist
--
  namespaceSelector:
    matchExpressions:
    - key: istio-injection
      operator: In
      values:
      - enabled
--
  namespaceSelector:
    matchExpressions:
    - key: istio-injection
      operator: DoesNotExist
    - key: istio.io/rev
      operator: DoesNotExist
```
#### Invoking the injection webhook for pods
      
          `kubectl get namespace -L istio-injection`
```
NAME              STATUS   AGE     ISTIO-INJECTION
default           Active   20d     enabled
istio-system      Active   3h25m   
kube-node-lease   Active   20d     
kube-public       Active   20d     
kube-system       Active   20d     
metallb-system    Active   20d    
```

#### Checking the default injection policy in the istio-sidecar-injector configmap

          `kubectl -n istio-system get configmap istio-sidecar-injector -o jsonpath='{.data.config}' | grep policy:`
policy: `enabled`

# Deploy ArgoCD directly using manifests

`kubectl create namespace argocd`
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

View Deployment

`kubectl get pods -n argocd`
For production preferably use Autopilot which commit all configs to git so ArgoCD can manage itself using GitOps


