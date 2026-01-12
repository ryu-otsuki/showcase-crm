# Showcase CRM - Claude Code設定

## プロジェクト概要
B2B顧客管理・商談パイプラインシステム。AI顧客スコアリング機能付き。

## 技術スタック
- **Frontend**: Next.js 15 (App Router), TypeScript, TailwindCSS, shadcn/ui
- **Backend**: Next.js API Routes, Prisma ORM
- **Database**: Supabase (PostgreSQL)
- **Auth**: Supabase Auth
- **AI**: Gemini API
- **Hosting**: Vercel

## ディレクトリ構造
```
crm/
├── src/
│   ├── app/           # Next.js App Router
│   ├── components/    # UIコンポーネント
│   ├── lib/           # ユーティリティ、API client
│   ├── hooks/         # カスタムフック
│   └── types/         # 型定義
├── prisma/            # Prismaスキーマ、マイグレーション
├── docs/              # 詳細ドキュメント
└── tests/             # テストファイル
```

## 開発コマンド
```bash
npm run dev          # 開発サーバー起動
npm run build        # プロダクションビルド
npm run test         # テスト実行
npm run lint         # ESLint実行
npm run type-check   # TypeScriptチェック
npm run db:migrate   # DBマイグレーション
npm run db:seed      # テストデータ投入
```

## 開発ルール

### コーディング規約
- TypeScript strict mode使用
- 関数コンポーネント + hooks
- 命名: camelCase（変数/関数）、PascalCase（コンポーネント/型）
- インポート順: React → 外部ライブラリ → 内部モジュール → 型

### Git規約
- コミットメッセージ: Conventional Commits形式
  - `feat:` 新機能
  - `fix:` バグ修正
  - `docs:` ドキュメント
  - `refactor:` リファクタリング
  - `test:` テスト
- ブランチ: `feature/issue-番号-説明` 形式

### API設計
- RESTful API
- レスポンス形式: `{ data: T } | { error: string }`
- バリデーション: Zod使用
- 認証: Supabase JWT

## 詳細ドキュメント
- [要件定義書](docs/REQUIREMENTS.md)
- [アーキテクチャ](docs/ARCHITECTURE.md)
- [データベース設計](docs/DATABASE.md)
- [API設計](docs/API.md)
- [Claude Code活用ガイド](docs/CLAUDE_CODE_GUIDE.md)

## 環境変数
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
GEMINI_API_KEY=
```

## 注意事項
- `.env.local`はコミット禁止
- Prismaスキーマ変更後は必ず`npm run db:migrate`実行
- APIルートには必ず認証チェックを実装
