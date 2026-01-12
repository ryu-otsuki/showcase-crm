# /deploy - Vercelデプロイ

Vercelへの本番デプロイを実行します。

## 実行内容

1. TypeScriptの型チェック
2. ESLintによるコード検証
3. テスト実行
4. プロダクションビルド
5. Vercelへデプロイ

## 使用方法

```
/deploy
```

## 実行コマンド

```bash
npm run type-check && npm run lint && npm run test && npm run build && vercel --prod
```

## 前提条件

- Vercel CLIがインストールされていること
- `vercel login`で認証済みであること
- プロジェクトがVercelにリンクされていること

## 注意事項

- デプロイ前に必ずローカルでビルドが通ることを確認してください
- mainブランチからのみデプロイを推奨します
