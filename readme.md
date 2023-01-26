# Kubernetesまとめ

Kubernetesの理解をまとめています。この内容は、以下の書籍の、第５章「Kubernetes入門」に沿って理解を進めています。書籍が古い(2018年)ため、一部用語が古いといったことがあるようです。（✖️ Master Node　→　◯ Control Plane）

[Docker/Kubernetes 実践コンテナ開発入門](https://gihyo.jp/book/2018/978-4-297-10033-9)
![book](./assets/img/book.jpg)

<br>

# 基本コマンド
- [こちら](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/)の公式チートシートが参考になりそう
  - kubectl config [Contextの設定](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#kubectl%E3%82%B3%E3%83%B3%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88%E3%81%AE%E8%A8%AD%E5%AE%9A)
  - kubectl apply/create/explain [Objectの作成](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#kubectl-apply)
  - kubectl get/describe [リソースの検索と閲覧](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E6%A4%9C%E7%B4%A2%E3%81%A8%E9%96%B2%E8%A6%A7)
  - kubectl set/rollout [リソースの更新](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88)
  - kubectl patch [リソースへのパッチ適用](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%B8%E3%81%AE%E3%83%91%E3%83%83%E3%83%81%E9%81%A9%E7%94%A8)
  - kubectl edit [リソースの編集](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E7%B7%A8%E9%9B%86)
  - kubectl scale [リソースのスケーリング](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E3%82%B9%E3%82%B1%E3%83%BC%E3%83%AA%E3%83%B3%E3%82%B0)
  - kubectl delete [リソースの削除](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E5%89%8A%E9%99%A4)
  - kubectl logs [実行中のポッドとの対話処理](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E5%AE%9F%E8%A1%8C%E4%B8%AD%E3%81%AE%E3%83%9D%E3%83%83%E3%83%89%E3%81%A8%E3%81%AE%E5%AF%BE%E8%A9%B1%E5%87%A6%E7%90%86)

<br>

- 特定の形式で端末ウィンドウに詳細を出力するには、サポートされているkubectlコマンドに[-o(または--output)フラグを追加](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/#%E5%87%BA%E5%8A%9B%E3%81%AE%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%83%E3%83%88s)する。
  ```sh
  # クラスター内で実行中のすべてのイメージ名を表示する
  kubectl get pods -A -o=custom-columns='DATA:spec.containers[*].image'

  # "k8s.gcr.io/coredns:1.6.2"を除いたすべてのイメージ名を表示する
  kubectl get pods -A -o=custom-columns='DATA:spec.containers[?(@.image!="k8s.gcr.io/coredns:1.6.2")].image'

  # 名前に関係なくmetadata以下のすべてのフィールドを表示する
  kubectl get pods -A -o=custom-columns='DATA:metadata.*'
  ```

<br>

# Kubernetesリソース
## 一覧

|リソース名|用途|(理解)|
|--|----|--|
|Pod|コンテナ集合体の単位でコンテナを実行する方法を定義する|済|
|Node|Kubernetesクラスタで実行するコンテナを配置するためのサーバ|済？|
|ReplicaSet|同じ仕様のPodを複数生成・管理する|済|
|StatefulSet|同じ仕様で一意性のあるPodを複数生成・管理する|
|DaemonSet|???|
|Deployment|ReplicaSetの世代管理をする|済|
|Service|Podの集合にアクセスするための経路を定義する|
|Ingress|ServiceをKubernetesクラスの外に公開する|
|Configmap|設定情報を定義し、Podに供給する|
|Secret|認証情報等の機密データを定義する|
|Job|常駐目的ではない複数のPodを作成し、正常終了することを保証する|
|CronJob|cron表記でスケジューリング|
|ClusterIp|???|

<br>

## 詳細

<br>

### Pod
- 単体または複数のコンテナを一纏めにした単位。~~Kubernetesでのデプロイ、スケーリングはこの単位で操作する。~~ (デプロイは、後述のDeploymentで操作する。Pod自体の定義は、最小単位と捉えるのが良さそう）
- 複数コンテナが密結合を望ましい構成の場合には、複数コンテナをPodとしてまとめる。WebサーバとWebAppサーバ等。

<br>

### Node
- Kubernetesのクラスタ下にあるリソースの集合体。<u>コンテナをデプロイするのに使用される</u>
- Kubernetesクラスタには、最低でも一つの「Control Plane」~~「Master Node」~~ が存在し、n個のControl Planeと、m個のそれ以外のNodeに分かれる。(Master Nodeというのは、古い呼び方らしい)
  ![cluster](./assets/img/cluster.png)
- Control Planeには、次の管理コンポーネントが存在する。
  |コンポーネント名|役割|
  |--|--|
  |kube-apiserver|KubernetesのAPIを公開するコンポーネント。kubetctlからのリソースの役割を受け付ける|
  |etcd|高可用性を備えた分散キーバリューストアで、Kubernetesクラスタのバッキングストアとして利用される|
  |kube-scheduler|Nodeを監視し、コンテナを配置する最適なNodeを選択する。|
  |kube-controller-manager|リソースを制御するコントローラーを実行する|
- 一般的には、Control Planeは３台配置して単一障害点とならないように構成するとのこと。  
------
#### 以下、課題・理解不足

  - <i><u>「デプロイするのに使われる」がよくわからない。Control Planeがそれを担う？それ以外のNode（Worker Node?）は、デプロイ先と解釈しているが違うのか？ </i></u>
  - <i><u>Master以外のノード(Worker Node?)と、後述のReplicaSet、Deploymentを使用したデプロイの関係がわからない。後述のリソースを読み進めると、Containerの集まりであるPodを、Deploymentを使ってデプロイすると理解している。が、ここの説明にNodeは出てこない。デプロイ先として指定するものではないのか？</i></u>  

    → あっている。Control Plane(AzureではAKSが担当)が、ワーカノードに対してデプロイを行う。デプロイ先は、各種yamlファイルにある[「Node Selector」で指定する](https://kubernetes.io/ja/docs/concepts/scheduling-eviction/assign-pod-node/)が指定する。例えば、GPUのプラットフォームに配置する場合や、専有(Dedecated)に配置するといった場合。指定しなければ、自動で配置される。
  - Master Nodeのコンポーネント内容は、ほとんど理解できていない。
  - AKSで構成した場合、このMaster NodeがAKSクラスタとなる、という認識であっているか？
  - AKSなりEKSで実際に構築してみないと、オンプレミスとの違いはわからない・・・。


<br>

### ReplicaSet
- <u> ReplicaSetはレプリカ数を制御する単位として把握すれば良い。実際には後述のDeploymentを使って世代管理を含めて操作することが多いため、ReplicaSet.yamlを直接操作する機会は少ない。 </u>
- ReplicaSet.yaml(ファイル名は任意)に構成を記述する。
  ``` yml
  apiVersion: apps/v1
  kind: ReplicaSet
  :
  spec:
    replicas: 3
    template: # template以下はPodリソースにおける定義と同じ
    :
    spec:
      containers:
      - name: nginx
        images: nginx:x.x.x
        env:
        :
  ```
  各種パラメタについては、[公式リファレンスを参照](https://kubernetes.io/ja/docs/concepts/workloads/controllers/replicaset/)

- 同じPodを指定した数だけ複製させるためのKubernetesリソース。
- 上記の`replicas`に、複製する個数を記載する。
- 適用コマンドは次の通り。
  > kubectl apply -f xxx_replicaset.yaml
- Podの数を減らすと、減らした分のPodは削除される。
- 以下のように、ReplicaSet自体を削除すると、関連するPodも削除される
  > kubectl delete -f xxx_replicaset.yaml

<br>

### Deployment
- アプリケーションをデプロイする際の基本単位となるKubernetesリソース。
- クラスタ化・冗長化構成をもったシステム全体に対し、世代管理によるデプロイ・ロールバックを行う。
- Deployment.yaml(ファイル名は任意)に構成を記述する。
  ``` yml
  apiVersion: apps/v1
  kind: Deployment  # ← ReplicaSetとはここが違う・・・。ぐらい。
  :
  spec:
    replicas: 3
    template: # template以下はPodリソースにおける定義と同じ
    :
    spec:
      containers:
      - name: nginx
        images: nginx:x.x.x
        env:
        :
  ```
  各種パラメタについては、[公式リファレンス](https://kubernetes.io/ja/docs/concepts/workloads/controllers/deployment/)を参照。

- <b><h4>デプロイ</h4></b>

  - 適用コマンドは次の通り。
    > kubectl apply -f xxx_reployment.yaml --record

  - `--record`を付与することによって、デプロイの履歴を以下のコマンドで確認できるようになる。
    > kubectl rollout history deployment [app_name]
    ```sh
    REVISION  CHANGE-CAUSE
    1         kubectl apply -f xxx_reployment.yaml --record=true
    ```

- <b><h4>デプロイ（更新）</h4></b> 

  - コマンドは`デプロイ`と同じ。
  - コンテナの定義、または構成を変更して再デプロイすることで、REVISIONがインクリメントされ、新しいPodがロードされていく。古いPodは段階的に停止されていく。
  - ymlの`replicas`のみを増減させた場合、指定された値に沿ってPodの増減は行われるが、REVISIONは変わらない。
    > kubectl apply -f xxx_deployment.yaml --record   # replicasを3→4へ変更

    ```sh
    NAME                    READY     STATUS                RESTARTS  AGE
    echo-5f978bc465-5vdnl   2/2       Running               0         6m
    echo-5f978bc465-9zjsa   2/2       Running               0         7m
    echo-5f978bc465-3e204   0/2       ContainerCreaeting    0         6m
    echo-5f978bc465-182b0   2/2       Running               0         7m
    ```

- <b><h4>ロールバック</h4></b>

  - 適用コマンドは次の通り `undo`。これにより、直前のリリース構成が復元される。
    > kubectl rollout undo deployment [app_name]

  - 事前にロールバックの内容を確認したい場合は、次のコマンドで内容を確認できる。
    > \# REVISIONの一覧を表示
    > kubectl rollout history deployment [app_name]
    > \# 指定したREVISIONの構成内容(deployment.yaml)を表示
    > kubectl rollout history deployment [app_name] --revision=1


<br>

## Azure（AKS）の関連サービス
- Node Pool
- Azure AD Identity
- Workload Identity
- Pod ID Identity
