---
weight: 4
title: "Live Update"
---

# Live Update

```shell
git clone https://github.com/vmjpsa/node-hello-tanzu.git
cd node-hello-tanzu
```

```shell
docker login $REPO_URL -u $REPO_USER -p $REPO_PASSWORD
```

```shell
cat <<EOF > Tiltfile
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
tanzu apps workload apply node-hello-tanzu  --local-path . -s $REPO_URL/tap-workshop/node-hello-tanzu-${SESSION_NUMBER} -y
```
```
Publishing source in "." to "$REPO_URL/tap-workshop/node-hello-tanzu-${SESSION_NUMBER}"...
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
      9 + |    image: $REPO_URL/tap-workshop/node-hello-tanzu-${SESSION_NUMBER}:latest@sha256:abcdef...
```


```shell
tilt up
```

```
coder@code-server-696f98d647-vx77m:~/node-hello-tanzu$ tilt up
Tilt started on http://localhost:10350/
v0.30.11, built 2022-11-07

(space) to open the browser
(s) to stream logs (--stream=true)
(t) to open legacy terminal mode (--legacy=true)
(ctrl-c) to exit
Tilt started on http://localhost:10350/
v0.30.11, built 2022-11-07

Initial Build
Loading Tiltfile at: /home/coder/node-hello-tanzu/Tiltfile
Successfully loaded Tiltfile (1.415443ms)
ERROR: [] "msg"="Reconciler error" "error"="Failed to update API server: create configmaps/node-hello-tanzu-disable: configmaps.tilt.dev \"node-hello-tanzu-disable\" already exists" "controller"="tiltfile" "controllerGroup"="tilt.dev" "controllerKind"="Tiltfile" "Tiltfile"={"name":"(Tiltfile)"} "namespace"="" "name"="(Tiltfile)" "reconcileID"="caf681ca-5e03-4d6f-9e76-c6e0468da938"
WARNING: Writing Kubernetes config: could not create any of the following paths: /run/user/1000/tilt-dev/tilt-default/cluster
node-hello-t… │ 
node-hello-t… │ Initial Build
node-hello-t… │ STEP 1/1 — Deploying
node-hello-t… │      Running cmd: tanzu apps workload apply -f config/workload.yaml --live-update --local-path . --source-image $REPO_URL/tap-workshop/node-hello-tanzu-01 --namespace tap-workshop-01 --yes >/dev/null && kubectl get workload node-hello-tanzu --namespace tap-workshop-01 -o yaml
node-hello-t… │      Objects applied to cluster:
node-hello-t… │        → node-hello-tanzu:workload
node-hello-t… │ 
node-hello-t… │      Step 1 - 1.88s (Deploying)
node-hello-t… │      DONE IN: 1.88s 
node-hello-t… │ 
node-hello-t… │ 
node-hello-t… │ Tracking new pod rollout (node-hello-tanzu-00001-deployment-66df8f56b6-2bp5q):
node-hello-t… │      ┊ Scheduled       - <1s
node-hello-t… │      ┊ Initialized     - (…) Pending
node-hello-t… │      ┊ Ready           - (…) Pending
node-hello-t… │ [queue-proxy] {"severity":"INFO","timestamp":"2022-12-05T11:37:01.427999526Z","logger":"queueproxy","caller":"queue/main.go:197","message":"Starting queue-proxy","commit":"3666ce7","knative.dev/key":"tap-workshop-01/node-hello-tanzu-00001","knative.dev/pod":"node-hello-tanzu-00001-deployment-66df8f56b6-2bp5q"}
```


```
......
node-hello-t… │ 1 File Changed: [index.js]
node-hello-t… │ Will copy 1 file(s) to container: [node-hello-tanzu-00002-deployment-5ffd767b87-78m56/workload]
node-hello-t… │ - '/home/coder/node-hello-tanzu/index.js' --> '/workspace/index.js'
node-hello-t… │   → Container node-hello-tanzu-00002-deployment-5ffd767b87-78m56/workload updated!
```

Ctrl+C でLiveUpdate を中断しましょう。
また、ログでターミナルが汚れていると思いますので、確認し終わったらclear してしまいましょう。


```shell
clear
```

