## Prerequisites:
Argo CD Installed

## Install Argo Workflows with Argo Events
Name: bootstrap-infra-argo \
URL: https://github.com/blevinson/temp \
Path: manifest/namespace-install \
Revision: mod-dev_v1

## Install Artillery Operator via Argo CD

Name: artillery-operator \
URL: https://github.com/blevinson/temp \
Path: artillery/config/default \
Revision: mod-dev_v1

### Load basic tests:
```bash
kubectl apply -k artillery/examples/basic-loadtest
```

## Check load test
```bash
kubectl get loadtests basic-test
```
You should observe:
```bash
NAME         COMPLETIONS   DURATION   AGE     ENVIRONMENT
basic-test   2/2           6m38s      7m33s   dev
```

## Setup Prometheus (If ot setup already):
```bash
kubectl apply --server-side -f hack/prom-pushgateway/kube-prometheus-main/manifests/setup
kubectl apply --server-side -f hack/prom-pushgateway/kube-prometheus-main/manifests/
```

# Add the prometheus pushgateway repo to helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# source: https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-pushgateway
# install the pushgateway
helm install prometheus-pushgateway --atomic --set serviceMonitor.enabled=true prometheus-community/prometheus-pushgateway

# port-forward the prometheus-pushgateway service to localhost
kubectl port-forward svc/prometheus-pushgateway 9091

## Open another terminal:
### Ensure the Artillery Operator is running on your cluster
```bash
kubectl -n artillery-operator-system get deployment.apps/artillery-operator-controller-manager
```
### You should see this:
```bash
# NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
# artillery-operator-controller-manager   1/1     1            1           27s
```
### Run the load test:

```bash
kubectl apply -k artillery/examples/published-metrics-loadtest
```
### You should observe this:
```bash
# configmap/test-script created
# loadtest.loadtest.artillery.io/test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3 created
```
### Ensure the test is running

```bash
kubectl get loadtests test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3
```
### You should observe this:
```bash
# NAME                                        COMPLETIONS   DURATION   AGE   ENVIRONMENT
# test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3   0/4           60s        62s   staging
```

### Find the load test's workers
```bash
kubectl describe loadtests test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3
```
### You should observe the following:
```bash
# Events:
#  Type    Reason     Age                    From                 Message
#  ----    ------     ----                   ----                 -------
#  Normal  Created    2m30s                  loadtest-controller  Created Load Test worker master job: test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3
#  Normal  Running    2m29s (x2 over 2m29s)  loadtest-controller  Running Load Test worker pod: test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3-2qmzv
#  Normal  Running    2m29s (x2 over 2m29s)  loadtest-controller  Running Load Test worker pod: test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3-cn99l
#  Normal  Running    2m29s (x2 over 2m29s)  loadtest-controller  Running Load Test worker pod: test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3-bsvgp
#  Normal  Running    2m29s (x2 over 2m29s)  loadtest-controller  Running Load Test worker pod: test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3-gk92x
```

### Viewing test report metrics on the Pushgateway
#### Navigating to the Pushgateway, in our case at http://localhost:9091, you'll see:

```bash
job="test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3 "
```
Clicking on a job matching a Pod name displays the test report metrics for a specific worker:

artillery_k8s_counter, includes counter based metrics like engine_http_responses, etc...
artillery_k8s_rates, includes rates based metrics like engine_http_request_rate, etc...
artillery_k8s_summaries, includes summary based metrics like engine_http_response_time_min, etc...

### Viewing aggregated test report metrics on Prometheus

In our case, we're running Prometheus in our K8s cluster, to access the dashboard we'll port-forward it to port 9090.

```bash
kubectl -n monitoring port-forward service/prometheus-k8s 9090
# Forwarding from 127.0.0.1:9090 -> 9090
# Forwarding from [::1]:9090 -> 9090
```
Navigating to the dashboard on http://localhost:9090/ we can view aggregated test report metrics for our Load Test across all workers.

Now enter into the search input field:

```bash
artillery_k8s_counters{load_test_id="test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3", metric="engine_http_requests"}
```

This displays engine_http_requests metric for Load Test test-378dbbbd-03eb-4d0e-8a66-39033a76d0f3.


## Notes 
### The following was the only modification from otb Argo Workflows:
#### Added the following to the config/rbac/role.yaml file:

```yaml
- apiGroups:
- ""
  resources:
- configmaps
  verbs:
- get
- list
- watch
- create
- update
- patch
- delete
```
