# terraform_docker_public

terraform実行コンテナになります。

## 参考サイト

[dockerでterraform環境を構築して、AWSにVPCを作る。](https://syoblog.com/terraform-environment/)

## 開発環境構築方法

### dockerソースファイル入手

```bash
git clone https://github.com/naritomo08/terraform_docker_public.git terraform
cd terraform
```

後にファイル編集などをして、git通知が煩わしいときは
作成したフォルダで以下のコマンドを入れる。

```bash
 rm -rf .git
```

### ソースフォルダ作成

```bash
mkdir source
→本ファイル内にmain.tfなどのソースファイルを置く。
```

### 環境変数ファイル作成

環境変数を取得するための値の取り方は割愛します。

```bash
vi .env

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
→AWSの場合、操作ユーザに対応した
 上記の情報を入れる。

ARM_SUBSCRIPTION_ID=
ARM_CLIENT_ID=
ARM_CLIENT_SECRET=
ARM_TENANT_ID=

→AWSの場合、操作ユーザに対応した
 上記の情報を入れる。

gcp管理画面からアクセスjsonキーを入手して、ソースファイルと同じ場所に置く
```

##　開発環境操作

### 開発環境コンテナ起動/設定再読み込み

```bash
docker-compose up -d
```

### 開発環境コンテナ停止

```bash
docker-compose stop
```

### 開発環境コンテナ破棄

```bash
docker-compose down
```

### terraformコンテナログイン

```bash
docker-compose exec terraform ash
```

## terraform操作

terraformコンテナ内で実施すること。
main.tfを作成したフォルダ内で行うこと。

### ワークスペース初期化

```bash
terraform init
```

### 実行計画確認

```bash
terraform plan
```

### リソース作成

```bash
terraform apply
→yesを入力する。
```

### 結果確認

```bash
terraform show
```

### 作成したモジュールの削除

terraformで作成したモジュールは
必ず本コマンドで削除すること。

```bash
terraform destroy
→yesを入力する。
```

## 作成例

予め環境変数作成まで行っていること。
gcpに関してはダウンロードしたjsonファイルをgcp.jsonにして
ソースと同じ場所に置く。

### main.tf作成

sourceフォルダ内で以下の内容のファイルを作成する。

```bash
cd source
vi main.tf
```

#### AWS(VPC/サブネット作成)

```bash
#AWSを今回は使いますと言う宣言
provider "aws" {
  region = "ap-northeast-1"
}

#VPCのCIDERを記載します。
variable "vpc_cidr" {
  default = "192.168.0.0/23"
}

#下記のresourceで使うタグ用の変数を定義
variable "app_name" {
  default = "awsvpc"
}

#resourceでVPCを作成します。
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = var.app_name
  }
}

#上記のVPCにサブネットを追加します。
resource "aws_subnet" "public_1a" {
  # 上記で作成したVPCを参照し、そのVPC内にSubnetを作成します。
  vpc_id = "${aws_vpc.main.id}"

  # Subnetを作成するAZ
  availability_zone = "ap-northeast-1a"
  cidr_block        = "192.168.0.0/25"
  tags = {
    Name = "awsvpc-prod"
  }
}
```

#### Azure(リソースグループ作成)

```bash
provider azurerm {
    features {}
}

variable "resource_group_name" {
    type = string
    default = "AzureResource"
    description = ""
}

variable "location" {
    type = string
    default = "japaneast"
    description = ""
}

resource "azurerm_resource_group" "resource_group" {
    name = var.resource_group_name
    location = var.location
}
```

#### GCP(VPC作成)

```bash
provider "google" {
  credentials = file("gcp.json")
  project     = "<GCPプロジェクト名>"
  region      = "asia-northeast1"
  zone    = "asia-northeast1-a"
}

resource "google_compute_network" "vpc_network" {
  name = "gcpvpc"
}

```

### リソース作成

```bash
docker-compose up -d
docker-compose exec terraform ash
terraform init
terraform plan
terraform apply
ctrl+c
→コンソールで作成されていることを確認。
```

### リソース削除

terraformで作成したものは手動ではなく、
terraformで削除すること。

```bash
docker-compose exec terraform ash
terraform destroy
ctrl+c
→コンソールで削除されていることを確認。
```

### 後始末

```bash
docker-compose down
sudo rm -rf terraform
```
