# try-aws-eks

AWS EKS を試してみる。  
EKS on Fargate で nginx サーバを起動してみる。  

## ツール インストール

eksctl, kubectl をインストール。  
Homebrew で eksctl を入れれば kubectl も一緒に入る。  

```sh
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
```

## ALB Ingress Controller IAM ポリシー 作成

ALB Ingress Controller IAM ポリシーを作成。  

```sh
aws iam create-policy \
  --policy-name ALBIngressControllerIAMPolicy \
  --policy-document "$(curl https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json)"
```

## クラスタ 作成

クラスタを作成。  
10〜20分くらいかかる。  

```sh
eksctl create cluster -f cluster.yml
```

クラスタロール, クラスタロールバインディングを作成。  

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```

ALB Ingress Controller をデプロイ。  
`--cluster-name`, `--aws-vpc-id`, `--aws-region` を追加。  

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml
kubectl edit deployment.apps/alb-ingress-controller -n kube-system
```

```yml
spec:
  containers:
  - args:
    - --ingress-class=alb
    - --cluster-name=try-aws-eks
    - --aws-vpc-id=vpc-XXXXXXXXXXXXXXXXX
    - --aws-region=ap-northeast-1
```

## クラスタ 削除

後始末。  
10〜20分くらいかかる。  

```sh
eksctl delete cluster -f cluster.yml -w
```
