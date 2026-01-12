# データベース設計書

## 1. ER図

```
┌─────────────────┐       ┌─────────────────┐
│  Organization   │       │      User       │
├─────────────────┤       ├─────────────────┤
│ id              │◀──┐   │ id              │
│ name            │   │   │ email           │
│ plan            │   │   │ name            │
│ created_at      │   └───│ organization_id │
└─────────────────┘       │ role            │
         │                │ created_at      │
         │                └─────────────────┘
         │                         │
         ▼                         │
┌─────────────────┐                │
│    Customer     │                │
├─────────────────┤                │
│ id              │                │
│ organization_id │                │
│ name            │◀───────────────┘ (assigned_to)
│ industry        │
│ size            │
│ status          │
│ assigned_to     │
│ created_at      │
└─────────────────┘
         │
         ├──────────────────────────┐
         ▼                          ▼
┌─────────────────┐       ┌─────────────────┐
│    Contact      │       │      Deal       │
├─────────────────┤       ├─────────────────┤
│ id              │       │ id              │
│ customer_id     │       │ customer_id     │
│ name            │       │ name            │
│ title           │       │ amount          │
│ email           │       │ stage           │
│ phone           │       │ probability     │
│ is_primary      │       │ expected_close  │
│ created_at      │       │ assigned_to     │
└─────────────────┘       │ created_at      │
                          └─────────────────┘
                                   │
                                   ▼
                          ┌─────────────────┐
                          │    Activity     │
                          ├─────────────────┤
                          │ id              │
                          │ deal_id         │
                          │ customer_id     │
                          │ type            │
                          │ description     │
                          │ scheduled_at    │
                          │ completed_at    │
                          │ created_by      │
                          └─────────────────┘
```

## 2. テーブル定義

### 2.1 Organization（組織）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | 組織ID |
| name | VARCHAR(255) | NOT NULL | 組織名 |
| plan | ENUM | DEFAULT 'free' | プラン（free/pro/enterprise） |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |
| updated_at | TIMESTAMP | | 更新日時 |

### 2.2 User（ユーザー）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | ユーザーID（Supabase Auth連携） |
| email | VARCHAR(255) | UNIQUE, NOT NULL | メールアドレス |
| name | VARCHAR(255) | NOT NULL | 氏名 |
| avatar_url | VARCHAR(500) | | アバター画像URL |
| organization_id | UUID | FK | 所属組織 |
| role | ENUM | DEFAULT 'member' | ロール（admin/manager/member） |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |

### 2.3 Customer（顧客）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | 顧客ID |
| organization_id | UUID | FK, NOT NULL | 組織ID |
| name | VARCHAR(255) | NOT NULL | 会社名 |
| industry | VARCHAR(100) | | 業種 |
| size | ENUM | | 企業規模（startup/small/medium/large/enterprise） |
| website | VARCHAR(500) | | WebサイトURL |
| address | TEXT | | 住所 |
| status | ENUM | DEFAULT 'lead' | ステータス（lead/active/inactive/churned） |
| assigned_to | UUID | FK | 担当者 |
| tags | TEXT[] | | タグ |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |
| updated_at | TIMESTAMP | | 更新日時 |

### 2.4 Contact（担当者）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | 担当者ID |
| customer_id | UUID | FK, NOT NULL | 顧客ID |
| name | VARCHAR(255) | NOT NULL | 氏名 |
| title | VARCHAR(100) | | 役職 |
| email | VARCHAR(255) | | メールアドレス |
| phone | VARCHAR(50) | | 電話番号 |
| is_primary | BOOLEAN | DEFAULT false | 主担当者フラグ |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |

### 2.5 Deal（商談）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | 商談ID |
| customer_id | UUID | FK, NOT NULL | 顧客ID |
| name | VARCHAR(255) | NOT NULL | 案件名 |
| amount | DECIMAL(15,2) | | 金額 |
| stage | ENUM | DEFAULT 'lead' | ステージ（lead/qualification/proposal/negotiation/won/lost） |
| probability | INTEGER | CHECK 0-100 | 成約確度（%） |
| expected_close_date | DATE | | 予定クローズ日 |
| actual_close_date | DATE | | 実際のクローズ日 |
| lost_reason | VARCHAR(255) | | 失注理由 |
| assigned_to | UUID | FK | 担当者 |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |
| updated_at | TIMESTAMP | | 更新日時 |

### 2.6 Activity（活動）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | 活動ID |
| customer_id | UUID | FK | 顧客ID |
| deal_id | UUID | FK | 商談ID |
| type | ENUM | NOT NULL | 種類（call/email/meeting/note） |
| subject | VARCHAR(255) | | 件名 |
| description | TEXT | | 内容 |
| scheduled_at | TIMESTAMP | | 予定日時 |
| completed_at | TIMESTAMP | | 完了日時 |
| created_by | UUID | FK, NOT NULL | 作成者 |
| created_at | TIMESTAMP | DEFAULT NOW() | 作成日時 |

### 2.7 CustomerScore（AIスコア）
| カラム | 型 | 制約 | 説明 |
|--------|------|------|------|
| id | UUID | PK | スコアID |
| customer_id | UUID | FK, UNIQUE | 顧客ID |
| score | INTEGER | CHECK 0-100 | スコア |
| factors | JSONB | | スコア要因 |
| calculated_at | TIMESTAMP | NOT NULL | 算出日時 |
| expires_at | TIMESTAMP | NOT NULL | 有効期限 |

## 3. インデックス設計

```sql
-- 顧客検索
CREATE INDEX idx_customers_organization ON customers(organization_id);
CREATE INDEX idx_customers_status ON customers(organization_id, status);
CREATE INDEX idx_customers_assigned ON customers(assigned_to);

-- 商談検索
CREATE INDEX idx_deals_customer ON deals(customer_id);
CREATE INDEX idx_deals_stage ON deals(stage);
CREATE INDEX idx_deals_assigned ON deals(assigned_to);
CREATE INDEX idx_deals_expected_close ON deals(expected_close_date);

-- 活動検索
CREATE INDEX idx_activities_customer ON activities(customer_id);
CREATE INDEX idx_activities_deal ON activities(deal_id);
CREATE INDEX idx_activities_scheduled ON activities(scheduled_at);

-- 全文検索
CREATE INDEX idx_customers_name_gin ON customers USING gin(to_tsvector('japanese', name));
```

## 4. Row Level Security (RLS)

```sql
-- 組織内のデータのみアクセス可能
CREATE POLICY "Users can only access their organization's customers"
ON customers
FOR ALL
USING (
  organization_id = (
    SELECT organization_id FROM users WHERE id = auth.uid()
  )
);

-- メンバーは担当顧客のみ編集可能
CREATE POLICY "Members can only edit assigned customers"
ON customers
FOR UPDATE
USING (
  assigned_to = auth.uid()
  OR EXISTS (
    SELECT 1 FROM users
    WHERE id = auth.uid()
    AND role IN ('admin', 'manager')
  )
);
```

## 5. Prisma スキーマ（抜粋）

```prisma
model Customer {
  id             String    @id @default(uuid())
  organizationId String    @map("organization_id")
  name           String
  industry       String?
  size           CompanySize?
  website        String?
  address        String?
  status         CustomerStatus @default(lead)
  assignedTo     String?   @map("assigned_to")
  tags           String[]
  createdAt      DateTime  @default(now()) @map("created_at")
  updatedAt      DateTime  @updatedAt @map("updated_at")

  organization   Organization @relation(fields: [organizationId], references: [id])
  assignedUser   User?        @relation(fields: [assignedTo], references: [id])
  contacts       Contact[]
  deals          Deal[]
  activities     Activity[]
  score          CustomerScore?

  @@map("customers")
}

enum CustomerStatus {
  lead
  active
  inactive
  churned
}

enum CompanySize {
  startup
  small
  medium
  large
  enterprise
}
```
