# k8s-playground

オンプレミスの小規模環境（仮想マシン 3 台: マスター 1 / ワーカー 2）で Kubernetes クラスターを構築し、Argo CD でアプリを継続デプロイするための最小構成リポジトリです。

現状、L2/LB（例: MetalLB）は未導入のため、Argo CD へのアクセスはポートフォワードを利用します。

## Features

- CNI は Calico
- GitOps は Argo CD
- サンプルアプリの環境差分は Kustomize のオーバーレイで管理

## Usage

### 1. クラスターの構築

環境に合わせてクラスターを構築してください。今回は Calico を利用し、Pod ネットワークの CIDR をデフォルトから変更しています。

```shell
### kubeadm による構築例（CIDR を指定）
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### 2. Calico の導入

Calico の導入は[公式ドキュメント](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)を参照してください。

本リポジトリにはネットワーク帯域（CIDR）を変更した `custom-resources.yaml` を用意しているため、以下で適用します。

```shell
### ネットワーク帯域を変更したカスタムリソースの導入
$ kubectl apply -f calico/custom-resources.yaml
```

### 3. Argo CD の導入

Argo CD の導入は[公式ドキュメント](https://argo-cd.readthedocs.io/en/stable/getting_started)を参照してください。

```shell
### 現状 L2/LB を導入していないため、公開方法の例として NodePort を利用します。
$ kubectl apply -f argocd/argocd-nodeport.yaml

### Argo CD の サンプル Application を作成
$ kubectl apply -f argocd/application/sample-app.yaml
```

## Kustomize の構成

- `sample-app/base` に Deployment・Service・Namespace などの共通定義
- `sample-app/overlays` で `replicas` といった差分をパッチ
