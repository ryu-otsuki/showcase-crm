# API設計書

## 1. API概要

- **ベースURL**: `/api/v1`
- **認証**: Bearer Token (Supabase JWT)
- **形式**: JSON
- **エラー形式**: RFC 7807 Problem Details

## 2. 共通仕様

### リクエストヘッダー
```
Authorization: Bearer <token>
Content-Type: application/json
```

### レスポンス形式
```typescript
// 成功
{
  "data": T,
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// エラー
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力値が不正です",
    "details": [
      { "field": "email", "message": "有効なメールアドレスを入力してください" }
    ]
  }
}
```

### ページネーション
```typescript
// リクエスト
GET /api/v1/customers?page=1&limit=20

// レスポンス
{
  "data": [...],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    }
  }
}
```

## 3. エンドポイント一覧

### 3.1 認証 (`/api/v1/auth`)

| Method | Path | 説明 |
|--------|------|------|
| POST | /auth/signup | ユーザー登録 |
| POST | /auth/login | ログイン |
| POST | /auth/logout | ログアウト |
| GET | /auth/me | 現在のユーザー情報 |
| PATCH | /auth/me | プロフィール更新 |

### 3.2 顧客 (`/api/v1/customers`)

| Method | Path | 説明 |
|--------|------|------|
| GET | /customers | 顧客一覧 |
| POST | /customers | 顧客作成 |
| GET | /customers/:id | 顧客詳細 |
| PATCH | /customers/:id | 顧客更新 |
| DELETE | /customers/:id | 顧客削除 |
| GET | /customers/:id/contacts | 担当者一覧 |
| POST | /customers/:id/contacts | 担当者追加 |
| GET | /customers/:id/deals | 商談一覧 |
| GET | /customers/:id/activities | 活動履歴 |
| GET | /customers/:id/score | AIスコア取得 |
| POST | /customers/:id/score/refresh | AIスコア再計算 |

### 3.3 商談 (`/api/v1/deals`)

| Method | Path | 説明 |
|--------|------|------|
| GET | /deals | 商談一覧 |
| POST | /deals | 商談作成 |
| GET | /deals/:id | 商談詳細 |
| PATCH | /deals/:id | 商談更新 |
| DELETE | /deals/:id | 商談削除 |
| PATCH | /deals/:id/stage | ステージ変更 |
| GET | /deals/:id/activities | 活動履歴 |

### 3.4 活動 (`/api/v1/activities`)

| Method | Path | 説明 |
|--------|------|------|
| GET | /activities | 活動一覧 |
| POST | /activities | 活動作成 |
| GET | /activities/:id | 活動詳細 |
| PATCH | /activities/:id | 活動更新 |
| DELETE | /activities/:id | 活動削除 |
| POST | /activities/:id/complete | 活動完了 |

### 3.5 ダッシュボード (`/api/v1/dashboard`)

| Method | Path | 説明 |
|--------|------|------|
| GET | /dashboard/summary | サマリー |
| GET | /dashboard/pipeline | パイプライン統計 |
| GET | /dashboard/forecast | 売上予測 |
| GET | /dashboard/performance | 担当者パフォーマンス |

### 3.6 AI (`/api/v1/ai`)

| Method | Path | 説明 |
|--------|------|------|
| POST | /ai/score | 顧客スコアリング |
| POST | /ai/suggest-action | 次アクション提案 |
| POST | /ai/analyze | 顧客分析 |

## 4. API詳細

### 4.1 顧客一覧取得

```
GET /api/v1/customers
```

**クエリパラメータ**
| パラメータ | 型 | 説明 |
|-----------|------|------|
| page | number | ページ番号（default: 1） |
| limit | number | 取得件数（default: 20, max: 100） |
| status | string | ステータスフィルター |
| assignedTo | string | 担当者フィルター |
| search | string | キーワード検索 |
| sortBy | string | ソート項目（default: createdAt） |
| sortOrder | string | ソート順（asc/desc） |

**レスポンス例**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "株式会社サンプル",
      "industry": "IT",
      "size": "medium",
      "status": "active",
      "assignedTo": {
        "id": "uuid",
        "name": "田中太郎"
      },
      "score": 85,
      "dealsCount": 3,
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    }
  }
}
```

### 4.2 顧客作成

```
POST /api/v1/customers
```

**リクエストボディ**
```json
{
  "name": "株式会社サンプル",
  "industry": "IT",
  "size": "medium",
  "website": "https://example.com",
  "address": "東京都渋谷区...",
  "assignedTo": "user-uuid",
  "tags": ["新規", "優良"]
}
```

**バリデーション（Zodスキーマ）**
```typescript
const createCustomerSchema = z.object({
  name: z.string().min(1).max(255),
  industry: z.string().max(100).optional(),
  size: z.enum(['startup', 'small', 'medium', 'large', 'enterprise']).optional(),
  website: z.string().url().optional(),
  address: z.string().optional(),
  assignedTo: z.string().uuid().optional(),
  tags: z.array(z.string()).optional(),
});
```

### 4.3 AIスコアリング

```
POST /api/v1/ai/score
```

**リクエストボディ**
```json
{
  "customerId": "uuid"
}
```

**レスポンス例**
```json
{
  "data": {
    "customerId": "uuid",
    "score": 85,
    "factors": {
      "engagementLevel": 90,
      "dealHistory": 80,
      "companySize": 75,
      "recentActivity": 95
    },
    "recommendation": "高い成約確度です。提案フェーズに進めることを推奨します。",
    "calculatedAt": "2024-01-01T00:00:00Z",
    "expiresAt": "2024-01-02T00:00:00Z"
  }
}
```

## 5. エラーコード

| コード | HTTPステータス | 説明 |
|--------|---------------|------|
| AUTH_REQUIRED | 401 | 認証が必要 |
| INVALID_TOKEN | 401 | トークンが無効 |
| FORBIDDEN | 403 | 権限不足 |
| NOT_FOUND | 404 | リソースが見つからない |
| VALIDATION_ERROR | 400 | バリデーションエラー |
| DUPLICATE_ENTRY | 409 | 重複エントリー |
| RATE_LIMITED | 429 | レート制限超過 |
| INTERNAL_ERROR | 500 | サーバーエラー |

## 6. レート制限

| プラン | 制限 |
|--------|------|
| Free | 100 req/min |
| Pro | 1000 req/min |
| Enterprise | 無制限 |

ヘッダーで残り回数を通知:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067200
```
