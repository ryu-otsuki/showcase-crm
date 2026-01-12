# /migrate - DBマイグレーション

Prismaを使用してデータベースマイグレーションを実行します。

## 実行内容

1. Prismaスキーマの検証
2. マイグレーションファイルの生成
3. データベースへの適用
4. Prisma Clientの再生成

## 使用方法

```
/migrate                    # 開発環境マイグレーション
/migrate --name [name]      # マイグレーション名を指定
/migrate --deploy           # 本番環境へ適用
/migrate --reset            # DBリセット（開発のみ）
```

## 実行コマンド

```bash
# 開発環境（対話的）
npx prisma migrate dev

# マイグレーション名指定
npx prisma migrate dev --name [name]

# 本番環境
npx prisma migrate deploy

# DBリセット（開発のみ）
npx prisma migrate reset
```

## スキーマファイル

```
prisma/
├── schema.prisma    # メインスキーマ
└── migrations/      # マイグレーション履歴
```

## 注意事項

- 本番環境での`--reset`は絶対に使用しないこと
- マイグレーション前にバックアップを取ることを推奨
- スキーマ変更後は必ず`npm run db:generate`でClientを更新
