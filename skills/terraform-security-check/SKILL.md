---
name: terraform-security-check
description: Terraformコードのセキュリティ観点でのレビューを行う。コード変更後や新規リソース追加時に使用。
---

# Terraform セキュリティチェック

対象ファイルのセキュリティリスクを以下の観点で確認し、問題点と修正案を提示する。

## チェック項目

### 1. 認証情報の露出

- `.tf` ファイルや `.tfvars` にアクセスキー、パスワード、トークンが直接記載されていないか
- 機密情報は Secrets Manager または SSM Parameter Store 参照になっているか

```hcl
# 悪い例
resource "aws_db_instance" "this" {
  password = "MyPassword123"
}

# 良い例
resource "aws_db_instance" "this" {
  manage_master_user_password = true
}
```

### 2. S3 パブリックアクセス

- `aws_s3_bucket_public_access_block` で4つすべて `true` になっているか

```hcl
resource "aws_s3_bucket_public_access_block" "this" {
  bucket                  = aws_s3_bucket.this.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### 3. セキュリティグループ

- インバウンドで `0.0.0.0/0` を許可しているルールがないか
- ALB → ECS, ECS → RDS のように送信元をセキュリティグループ参照にしているか

```hcl
# 悪い例
ingress {
  from_port   = 5432
  to_port     = 5432
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}

# 良い例
ingress {
  from_port       = 5432
  to_port         = 5432
  protocol        = "tcp"
  security_groups = [aws_security_group.ecs.id]
}
```

### 4. RDS・ElastiCache のアクセス制限

- `publicly_accessible = false` になっているか
- プライベートサブネットに配置されているか

### 5. IAM 最小権限

- `*` を使った過剰なアクション・リソース指定がないか
- ECS タスクロールは必要なアクションのみ許可しているか

```hcl
# 悪い例
statement {
  actions   = ["*"]
  resources = ["*"]
}

# 良い例
statement {
  actions = [
    "s3:GetObject",
    "s3:PutObject",
  ]
  resources = ["${aws_s3_bucket.app.arn}/*"]
}
```

### 6. 暗号化

- RDS の `storage_encrypted = true` が設定されているか
- S3 のサーバーサイド暗号化が設定されているか
- ElastiCache の `at_rest_encryption_enabled = true` が設定されているか

### 7. ECS コンテナの機密情報

- `environment` ブロックに直接パスワード等を記載していないか
- `secrets` ブロックで Secrets Manager / SSM から参照しているか

```hcl
# 良い例
secrets = [
  {
    name      = "DB_PASSWORD"
    valueFrom = "arn:aws:secretsmanager:ap-northeast-1:123456789:secret:myapp/db"
  }
]
```

## 出力形式

チェック結果は以下の形式で報告する。

```
## セキュリティチェック結果

### 問題あり
- [重大] <ファイル名>:<行番号> - <問題の説明>
  修正案: <修正後のコード例>

### 問題なし
- S3パブリックアクセスブロック: OK
- ...
```
