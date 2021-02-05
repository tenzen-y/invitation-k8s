# とりあえず動かしてみる

とりあえず何も考えずに以下の手順に従って wordpress を起動してみましょう．

## kind cluster の起動

以下のコマンドで K8s クラスタを起動します．

```shell
$ kind create cluster --config manifests/kind.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.20.2) 🖼 
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

以下のコマンドを実行して，STATUS 欄が全て ```Ready``` になるまで待ちます．

```shell
$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE    VERSION
kind-control-plane   Ready    control-plane,master   2m7s   v1.20.2
kind-worker          Ready    <none>                 94s    v1.20.2
kind-worker2         Ready    <none>                 88s    v1.20.2
kind-worker3         Ready    <none>                 95s    v1.20.2
```

## 準備

以下のおまじないを唱えてください．

```shell
$ kubectl apply -f manifests/ingress-nginx-controller
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
```

以下のコマンドの実行結果が ```SucceededSucceededRunning``` になるまで待つ

```shell
$ kubectl get pods -n ingress-nginx -l job-name=ingress-nginx-admission-create -ojsonpath={.items[*].status.phase} && kubectl get pods -n ingress-nginx -l job-name=ingress-nginx-admission-patch -ojsonpath={.items[*].status.phase} && kubectl get pods -n ingress-nginx -l app.kubernetes.io/component=controller -ojsonpath={.items[*].status.phase} && echo

SucceededSucceededRunning
```

## mysql の起動

以下のコマンドを実行して MySQL サーバを起動します．

```shell
$ kubectl apply -f manifests/mysql/pvc.yaml
$ kubectl apply -f manifests/mysql/secret.yaml
$ kubectl apply -f manifests/mysql/service.yaml
$ kubectl apply -f manifests/mysql/deployment.yaml
```

## wordpress の起動

以下のコマンドを実行して WordPress サーバを起動します．

```shell
$ kubectl apply -f manifests/wordpress/configmap.yaml
$ kubectl apply -f manifests/wordpress/pvc.yaml
$ kubectl apply -f manifests/wordpress/service.yaml
$ kubectl apply -f manifests/wordpress/deployment.yaml
$ kubectl apply -f manifests/wordpress/ingress.yaml
```

## 確認

以下のコマンドを実行して 2 つの Pod の ```STATUS``` が ```Running``` になっているか確認します．

```shell
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-5994d99f46-pvw2n        1/1     Running   0          74s
wordpress-mysql-6c479567b-vrmb8   1/1     Running   0          2m51s
```

ブラウザで ```http://localhost/``` にアクセスします．成功していると以下のように表示されます．

[wordpress](./wordpress.png)
