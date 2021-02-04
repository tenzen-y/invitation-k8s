# start で作った wordpress を編集してみる

## コンテナの個数

```manifest/wordpress/deployment.yaml``` の ```replicas``` を ```1``` から ```3``` に書き換える．

```diff
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
-  replicas: 1
+  replicas: 3
```

以下のコマンドを実行して反映する

```shell
$ kubectl apply -f manifests/wordpress/deployment.yaml
deployment.apps/wordpress configured
```

wordpress Pod が 3つになっていることを確認して，ブラウザで正常動作を確認する．

```shell
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-5994d99f46-5tjc9        1/1     Running   0          18s
wordpress-5994d99f46-6sqnt        1/1     Running   0          3s
wordpress-5994d99f46-f7gxq        1/1     Running   0          3s
wordpress-mysql-6c479567b-2l9hn   1/1     Running   0          22s
```

