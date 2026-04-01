---
name: terraform-module-generate
description: Terraformモジュールのひな形を生成する。新規モジュール作成時に使用。
---

# Terraform モジュール生成

新規 Terraform モジュールのひな形を生成する。

## 手順

1. `modules/<module-name>/` ディレクトリを作成する
2. 以下の4ファイルを生成する

### versions.tf

```hcl
terraform {
  required_version = ">= 1.9"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### variables.tf

```hcl
variable "project_name" {
  description = "プロジェクト名"
  type        = string
}

variable "environment" {
  description = "環境名 (dev / stg / prod)"
  type        = string
}

variable "owner" {
  description = "担当チームまたは個人"
  type        = string
}

# --- モジュール固有の変数をここに追記 ---
```

### outputs.tf

```hcl
# --- 出力値をここに定義 ---
# 例:
# output "id" {
#   description = "リソースのID"
#   value       = aws_xxx_yyy.this.id
# }
```

### main.tf

```hcl
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    Owner       = var.owner
  }
  name_prefix = "${var.project_name}-${var.environment}"
}

# --- リソース定義をここに追記 ---
```

## 注意事項

- モジュール名は `snake_case` で命名する
- AWSリソースの `name` 属性は `"${local.name_prefix}-<role>"` の形式にする
- 機密情報は変数に持たせず、呼び出し元で Secrets Manager 参照を渡す
