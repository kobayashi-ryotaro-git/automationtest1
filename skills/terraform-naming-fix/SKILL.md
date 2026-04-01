---
name: terraform-naming-fix
description: Terraformコードの命名規則違反を検出して修正する。コードレビュー時や既存コードのリファクタリング時に使用。
---

# Terraform 命名規則チェック・修正

対象ファイルの命名規則違反を検出し、修正案を提示する。

## 命名規則

| 対象 | 規則 | 良い例 | 悪い例 |
|------|------|--------|--------|
| リソースラベル | `snake_case` | `aws_s3_bucket.app_logs` | `aws_s3_bucket.appLogs` |
| 変数名 | `snake_case` | `var.instance_type` | `var.instanceType` |
| ローカル変数 | `snake_case` | `local.common_tags` | `local.commonTags` |
| モジュール呼び出し | `snake_case` | `module.ecs_service` | `module.ecsService` |
| データソース | `snake_case` | `data.aws_vpc.main` | `data.aws_vpc.Main` |
| AWSリソース name 属性 | `<project>-<env>-<role>` | `"myapp-prod-alb"` | `"alb"`, `"MyAppALB"` |

## チェック手順

1. リソースラベル・変数名・ローカル変数・モジュール名に `camelCase` や `PascalCase` が使われていないか確認する
2. AWS リソースの `name` 属性が変数またはローカル変数を使用しているか確認する（文字列直打ち禁止）
3. `name` 属性が `"${local.name_prefix}-<role>"` または `"${var.project_name}-${var.environment}-<role>"` の形式になっているか確認する

## 修正例

```hcl
# 修正前（違反例）
locals {
  commonTags = {
    Project = var.projectName
  }
}

resource "aws_alb" "MyALB" {
  name = "ALB"
}

# 修正後
locals {
  common_tags = {
    Project = var.project_name
  }
}

resource "aws_alb" "this" {
  name = "${var.project_name}-${var.environment}-alb"
}
```

## 出力形式

```
## 命名規則チェック結果

### 違反箇所
- <ファイル名>:<行番号> - `<違反している名前>` → `<修正案>`

### 修正後のコード
<修正済みコードブロック>
```

## 注意事項

- 変数名変更は `variables.tf` だけでなく参照箇所もすべて変更する
- モジュールのラベル変更は `module.<name>` の参照箇所も合わせて変更する
- `terraform plan` で破壊的変更が発生しないか確認してから適用する
