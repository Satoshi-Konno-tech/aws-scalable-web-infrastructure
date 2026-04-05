# Scalable Web Architecture on AWS (ALB + Auto Scaling)

## 概要

AWS上にWebサーバー環境を構築し、Application Load Balancer (ALB) を利用してEC2インスタンスへトラフィックをルーティングする構成を作成しました。

セキュリティ設計として、EC2インスタンスにはパブリックIPを付与せず、Security GroupによりHTTP通信はALB経由のみ許可することで、外部からの直接アクセスを制御しています。

また、本構成ではAuto Scaling Group（ASG）を導入し、負荷に応じてインスタンス数が自動的に増減することを検証しました。

---

## アーキテクチャ

Internet  
↓  
Application Load Balancer  
↓  
Target Group  
↓  
EC2（Auto Scaling Group / Private Subnet）  
↑  
Bastion（SSH接続）

---

## 使用サービス

- Amazon VPC
- Subnet
- Internet Gateway
- Route Table
- Security Group
- Amazon EC2
- Apache HTTP Server
- Application Load Balancer
- Target Group
- Auto Scaling Group
- Launch Template
- Amazon CloudWatch

---

## ネットワーク構成

### VPC

CIDR

10.0.0.0/16

---

### Public Subnet

10.0.1.0/24  
10.0.2.0/24  

用途:
- ALB配置
- Bastionサーバ配置

---

### Private Subnet

- EC2（ASG配下）を配置
- パブリックIPなし

---

### Route Table

0.0.0.0/0  
↓  
Internet Gateway  

---

## EC2 Web Server

### インスタンス

- Amazon Linux 2023
- t3.micro

---

### Apacheインストール

- sudo dnf install httpd -y
- sudo systemctl start httpd
- sudo systemctl enable httpd


---

### テストページ

Hello ASG  

※Auto Scaling環境であることを明示するため表示内容を変更

---

## Application Load Balancer

### Scheme

Internet-facing

### Listener

HTTP : 80

---

## Target Group

### 基本設定

- Name: portfolio-web-tg
- Protocol: HTTP
- Port: 80
- Target type: Instance

---

### Health Check

- Protocol: HTTP
- Path: /

EC2インスタンスが Healthy 状態であることを確認

---

## Auto Scaling Group

### 設定

- 最小: 2
- 希望: 2
- 最大: 3

---

### 構成

- Launch Templateを使用
- 複数AZにインスタンスを配置

---

## セキュリティ設計

### ALB Security Group

- HTTP 80
- Source: 0.0.0.0/0

---

### EC2 Security Group

- SSH 22  
  Source: 自身のIPのみ

- HTTP 80  
  Source: ALB Security Group

---

### 通信フロー

Internet  
↓  
ALB  
↓  
EC2（Private）

EC2への直接アクセスを防止し、ALB経由のみ通信可能としています。

---

## 動作確認

### ALB DNSへアクセス

http://portfolio-alb-xxxx.ap-northeast-1.elb.amazonaws.com

---

### 表示

Hello ASG

---

## Auto Scaling 検証

負荷に応じたスケーリング動作を確認しました。

---

### スケールアウト

EC2内部でCPU負荷を発生

yes > /dev/null &
yes > /dev/null &


---

### 結果

2台 → 3台

- Auto Scaling Groupによりインスタンスが自動起動
- Target Groupに登録されHealthyになることを確認

---

### スケールイン

CPU負荷停止

- pkill yes


---

### 結果

3台 → 2台

- インスタンスが自動削除
- Target Groupから削除されることを確認

---

### 検証結果

CPU上昇 → スケールアウト（2→3）  
CPU低下 → スケールイン（3→2）

負荷に応じてインスタンス数が自動調整されることを確認しました。

---

## スクリーンショット

### Webアクセス（ALB経由）

![web](images/web.jpg)

---

### VPC

![vpc](images/vpc.jpg)

---

### Subnet

![subnet](images/subnet.jpg)

---

### Route Table

![route-table](images/route-table.jpg)

---

### Application Load Balancer

![alb](images/alb.jpg)

---

### Target Group

![targetgroup](images/targetgroup.jpg)

---

### Auto Scaling（スケールアウト）

![scale-out](images/scale-out.jpg)

---

### Auto Scaling（スケールイン）

![scale-in](images/scale-in.jpg)

---

## 学び

- VPCを用いたAWSネットワーク設計
- SubnetとRoute Tableの関係
- Internet Gatewayによるインターネット接続
- Application Load Balancerによるトラフィック分散
- Target GroupによるEC2へのルーティング
- ヘルスチェックによるサーバー状態監視
- Security Groupを用いたアクセス制御
- Auto Scalingによる可用性とコスト最適化
- CloudWatchによるスケーリング制御

---

## 今後の改善

- HTTPS化（ACM + ALB）
- TerraformによるInfrastructure as Code化
- CloudWatchアラームの詳細設計
- ログ監視（CloudWatch Logs）