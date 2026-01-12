# アーキテクチャ設計書

## 1. システム構成図

```
┌─────────────────────────────────────────────────────────────┐
│                         Client                               │
│                    (Next.js Frontend)                        │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                    Vercel Edge                               │
│              (Middleware: Auth Check)                        │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
┌───────────────┐           ┌───────────────┐
│  Server       │           │  API Routes   │
│  Components   │           │  (REST API)   │
└───────┬───────┘           └───────┬───────┘
        │                           │
        └─────────────┬─────────────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
┌───────────┐  ┌───────────┐  ┌───────────┐
│ Supabase  │  │ Supabase  │  │ Gemini    │
│ PostgreSQL│  │ Auth      │  │ API       │
└───────────┘  └───────────┘  └───────────┘
```

## 2. レイヤー構成

### 2.1 プレゼンテーション層
```
src/app/              # ページコンポーネント (Server/Client)
src/components/
├── ui/               # shadcn/ui基本コンポーネント
├── features/         # 機能別コンポーネント
│   ├── customers/    # 顧客管理
│   ├── deals/        # 商談管理
│   ├── dashboard/    # ダッシュボード
│   └── auth/         # 認証
└── layouts/          # レイアウトコンポーネント
```

### 2.2 アプリケーション層
```
src/lib/
├── actions/          # Server Actions
├── api/              # APIクライアント
├── validations/      # Zodスキーマ
└── utils/            # ユーティリティ

src/hooks/            # カスタムフック
├── use-customers.ts
├── use-deals.ts
└── use-auth.ts
```

### 2.3 データアクセス層
```
src/lib/
├── db/               # Prismaクライアント
├── repositories/     # リポジトリパターン
│   ├── customer.ts
│   ├── deal.ts
│   └── activity.ts
└── services/         # ビジネスロジック
    ├── scoring.ts    # AIスコアリング
    └── analytics.ts  # KPI計算
```

## 3. 認証・認可フロー

```
┌────────┐    ┌──────────┐    ┌────────────┐    ┌──────────┐
│ Client │───▶│Middleware│───▶│ Supabase   │───▶│ Database │
│        │    │(JWT検証) │    │ Auth       │    │ (RLS)    │
└────────┘    └──────────┘    └────────────┘    └──────────┘
     │              │                                  │
     │              ▼                                  │
     │        ┌──────────┐                            │
     │        │ Role     │                            │
     │        │ Check    │                            │
     │        └──────────┘                            │
     │              │                                  │
     └──────────────┴──────────────────────────────────┘
```

### ロール定義
| ロール | 権限 |
|--------|------|
| admin | 全機能アクセス、チーム管理 |
| manager | チーム全体の閲覧・編集 |
| member | 担当顧客のみ閲覧・編集 |

## 4. データフロー

### 4.1 顧客スコアリング
```
1. 顧客データ取得
   └─▶ Prisma → PostgreSQL

2. コンテキスト生成
   └─▶ 商談履歴、活動履歴、企業情報を集約

3. AI分析
   └─▶ Gemini API → スコア算出

4. スコア保存
   └─▶ CustomerScore テーブルに保存

5. キャッシュ
   └─▶ 24時間キャッシュ（再計算抑制）
```

### 4.2 ダッシュボードKPI
```
1. リアルタイムデータ
   └─▶ Server Component で直接DBクエリ

2. 集計データ
   └─▶ PostgreSQL Materialized View

3. クライアント更新
   └─▶ React Query (5分間隔)
```

## 5. 状態管理

| 種類 | 技術 | 用途 |
|------|------|------|
| サーバー状態 | React Query | API データのキャッシュ |
| クライアント状態 | Zustand | UI状態、フィルター |
| フォーム状態 | React Hook Form | フォーム入力 |
| URL状態 | nuqs | 検索パラメータ |

## 6. エラーハンドリング

### API エラー形式
```typescript
type ApiResponse<T> =
  | { data: T; error: null }
  | { data: null; error: { code: string; message: string } };
```

### エラーコード
| コード | 説明 |
|--------|------|
| AUTH_REQUIRED | 認証が必要 |
| FORBIDDEN | 権限不足 |
| NOT_FOUND | リソースが見つからない |
| VALIDATION_ERROR | バリデーションエラー |
| INTERNAL_ERROR | サーバーエラー |

## 7. パフォーマンス最適化

### 7.1 フロントエンド
- Server Components でのデータフェッチ
- 動的インポート（lazy loading）
- 画像最適化（next/image）
- フォント最適化（next/font）

### 7.2 バックエンド
- Prisma の select/include 最適化
- インデックス設計
- コネクションプーリング（Supabase）
- エッジでのキャッシュ

### 7.3 データベース
- 複合インデックス
- Materialized View（集計）
- パーティショニング（将来）
