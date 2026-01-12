# /seed - テストデータ投入

開発・テスト用のサンプルデータをデータベースに投入します。

## 実行内容

1. 既存のテストデータをクリア（オプション）
2. サンプル組織・ユーザーの作成
3. サンプル顧客データの作成
4. サンプル商談データの作成
5. サンプル活動履歴の作成

## 使用方法

```
/seed              # テストデータ投入
/seed --clean      # クリアしてから投入
/seed --minimal    # 最小限のデータのみ
```

## 実行コマンド

```bash
# 基本
npx prisma db seed

# クリアしてから投入
npx prisma migrate reset && npx prisma db seed
```

## シードデータ

```typescript
// prisma/seed.ts

// 組織
- Demo Organization

// ユーザー
- admin@example.com (管理者)
- manager@example.com (マネージャー)
- member@example.com (メンバー)

// 顧客: 20社
// 商談: 50件
// 活動: 100件
```

## 注意事項

- 本番環境では絶対に実行しないこと
- 既存データが上書きされる可能性あり
- `--clean`オプションは全データを削除します
