---
weight: 4
title: "Live Update"
---

# Live Update

```shell
git clone https://github.com/vmjpsa/node-hello-tanzu.git
```

```shell
cat <<EOF > 
SOURCE_IMAGE = os.getenv("SOURCE_IMAGE", default='${REPO_URL}/tap-workshop/node-hello-tanzu-${SESSION_NUMBER}') # REPO_URL like harbor.tanzu.lab/tap-workshop/node-hello-tanzu
LOCAL_PATH = os.getenv("LOCAL_PATH", default='.')
NAMESPACE = os.getenv("NAMESPACE", default='')
K8S_TEST_CONTEXT = os.getenv("K8S_TEST_CONTEXT", default='')

allow_k8s_contexts(K8S_TEST_CONTEXT)

k8s_custom_deploy(
    'node-hello-tanzu',
    apply_cmd="tanzu apps workload apply -f config/workload.yaml --live-update" +
        " --local-path " + LOCAL_PATH +
        " --source-image " + SOURCE_IMAGE +
        " --namespace " + NAMESPACE +
        " --yes >/dev/null" +
        " && kubectl get workload node-hello-tanzu --namespace " + NAMESPACE + " -o yaml",
    delete_cmd="tanzu apps workload delete -f config/workload.yaml --namespace " + NAMESPACE + " --yes" ,
    deps=['.'],
    container_selector='workload',
    live_update=[
        fall_back_on(['package.json']),
        sync('.', '/workspace')
]
)

k8s_resource('node-hello-tanzu', port_forwards=["8080:8080"],
    extra_pod_selectors=[{'carto.run/workload-name': 'node-hello-tanzu', 'app.kubernetes.io/component': 'run'}])

allow_k8s_contexts('${CLUSTER_NAME}')

EOF


```

```shell
tanzu apps workload apply node-hello-tanzu  --local-path . -s harbor.vmjpsa.com/tap-workshop/node-hello-tanzu-${SESSION_NUMBER} -y
```
```
Publishing source in "." to "harbor.vmjpsa.com/tap-workshop/node-hello-tanzu-01"...
Published source
Create workload:
      1 + |---
      2 + |apiVersion: carto.run/v1alpha1
      3 + |kind: Workload
      4 + |metadata:
      5 + |  name: node-hello-tanzu
      6 + |  namespace: tap-workshop-01
      7 + |spec:
      8 + |  source:
      9 + |    image: harbor.vmjpsa.com/tap-workshop/node-hello-tanzu-01:latest@sha256:abcdef...
```


```shell
echo tilt up
```