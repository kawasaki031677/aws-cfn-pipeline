# aws-cfn-pipeline

このリポジトリは、AWS CloudFormation を使って CodePipeline ベースの CI/CD パイプラインを IaC（Infrastructure as Code）で構築するための日本語ドキュメント付きサンプルです。

- シングルアカウント構成
- クロスアカウント構成（別 AWS アカウントへのデプロイ）
- CodeCommit / CodeBuild / CodePipeline / EventBridge / IAM の基本構成
- 自動デプロイ用のビルド・デプロイスクリプト（buildspec）
- デプロイ対象のサンプルアプリケーション IaC（sample-iac）

---

## 目次

1. [概要](#概要)
2. [構成](#構成)
3. [利用方法](#利用方法)
4. [リソース](#リソース)
5. [注意事項](#注意事項)

---

## 概要

- 本プロジェクトは、AWS 上に継続的デプロイパイプラインを CloudFormation テンプレートで実装するためのベースラインを提供します。
- CloudFormation を使用して CodePipeline を構築し、ソースコードの変更をトリガーとした自動デプロイを実現します。
- 実運用導入前にテンプレートをレビューし、IAM ポリシー・権限・条件を自社ルールに適合させる必要があります。

---

## 構成

```
aws-pipeline/
├── README.md
├── pipelines/
│   ├── single-account/
│   │   ├── pipeline.yml
│   │   ├── iam.yml
│   │   └── eventbridge.yml
│   └── cross-account/
│       ├── pipeline.yml
│       ├── source.yml
│       ├── iam.yml
│       └── eventbridge.yml
├── buildspec/
│   ├── buildspec-build.yml
│   └── buildspec-deploy.yml
└── sample-iac/
    ├── parameters/
    │   ├── dev/
    │   │   ├── common.json
    │   │   └── sns.json
    │   └── prd/
    │       ├── common.json
    │       └── sns.json
    └── stacks/
        ├── iam.yml
        └── sns.yml
```

### pipelines

CodePipeline を作成する CloudFormation テンプレートです。

* `single-account`
  同一アカウント内で動作するパイプラインのサンプル

* `cross-account`
  別アカウントへデプロイするクロスアカウントパイプラインのサンプル

### buildspec

CodeBuild で使用するビルド・デプロイスクリプトです。自動デプロイを実現するための設定が含まれています。

* `buildspec-build.yml`
  ビルドフェーズのスクリプト

* `buildspec-deploy.yml`
  デプロイフェーズのスクリプト

### sample-iac

デプロイ対象となるサンプルアプリケーションの IaC です。実際のデプロイで使用される CloudFormation テンプレートとパラメータファイルが含まれています。

* `parameters/`
  環境別パラメータファイル（dev, prd）

* `stacks/`
  CloudFormation スタックテンプレート（IAM, SNS など）

---

## 利用方法

### 1. シングルアカウントパイプライン

```sh
aws cloudformation deploy \
  --template-file ./pipelines/single-account/pipeline.yml \
  --stack-name pipeline-stack \
  --parameter-overrides EnvironmentName=prd SystemName=example \
  --capabilities CAPABILITY_NAMED_IAM
```

### 2. クロスアカウントパイプライン

#### a. パイプラインアカウント

```sh
aws cloudformation deploy \
  --template-file ./pipelines/cross-account/pipeline.yml \
  --stack-name pipeline-stack \
  --parameter-overrides EnvironmentName=prd SystemName=example SourceAccountId=<SOURCE_ACCOUNT_ID> SourceAccountSystemName=source \
  --capabilities CAPABILITY_NAMED_IAM
```

#### b. ソースアカウント

```sh
aws cloudformation deploy \
  --template-file ./pipelines/cross-account/source.yml \
  --stack-name source-stack \
  --parameter-overrides EnvironmentName=prd SystemName=source PipelineAccountId=<PIPELINE_ACCOUNT_ID> PipelineAccountSystemName=example \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## リソース

- CodeCommit リポジトリ
- CodeBuild プロジェクト
- CodePipeline ステージ（ソース→ビルド→デプロイ）
- EventBridge ルール
- IAM ロールとポリシー

---

## 注意事項

- サンプルのまま本番環境にデプロイしないでください。
- 既存リソースとの名前衝突、権限の過剰付与を避けるため、テンプレートとパラメータを必ず見直してください。
- `eventbridge`, `iam`, `s3`, `kms` などの権限は最小権限ポリシーに調整してください。

---
