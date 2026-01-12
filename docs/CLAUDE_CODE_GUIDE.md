# Claude Code 活用ガイド

このドキュメントでは、本プロジェクトでのClaude Code機能の活用方法を説明します。

## 1. MCP (Model Context Protocol)

MCPを使用して外部サービスと連携します。

### 1.1 設定ファイル

`.claude/mcp.json`:
```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": ["-y", "@supabase/mcp-server"],
      "env": {
        "SUPABASE_URL": "${SUPABASE_URL}",
        "SUPABASE_SERVICE_ROLE_KEY": "${SUPABASE_SERVICE_ROLE_KEY}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-filesystem", "./src"]
    }
  }
}
```

### 1.2 Supabase MCP の活用

Supabase MCPを使うと、Claude Codeから直接データベース操作ができます。

**利用例:**
- テーブル構造の確認
- テストデータの投入
- クエリの実行・デバッグ
- RLSポリシーの確認

### 1.3 Filesystem MCP の活用

プロジェクトファイルの効率的な探索・編集が可能です。

## 2. スキル（カスタムコマンド）

### 2.1 スキル定義

`.claude/skills/` ディレクトリにスキルを定義します。

#### /deploy スキル
`.claude/skills/deploy/SKILL.md`:
```markdown
# Deploy Skill

Vercelへのデプロイを実行します。

## 実行内容
1. ビルドチェック
2. テスト実行
3. Vercelデプロイ

## コマンド
\`\`\`bash
npm run build && npm run test && vercel --prod
\`\`\`
```

#### /test スキル
`.claude/skills/test/SKILL.md`:
```markdown
# Test Skill

テストを実行します。

## オプション
- `--coverage`: カバレッジレポート生成
- `--watch`: ウォッチモード

## コマンド
\`\`\`bash
npm run test
\`\`\`
```

#### /migrate スキル
`.claude/skills/migrate/SKILL.md`:
```markdown
# Migrate Skill

Prismaマイグレーションを実行します。

## 実行内容
1. スキーマ検証
2. マイグレーション生成
3. マイグレーション適用

## コマンド
\`\`\`bash
npx prisma migrate dev
\`\`\`
```

#### /seed スキル
`.claude/skills/seed/SKILL.md`:
```markdown
# Seed Skill

テストデータを投入します。

## コマンド
\`\`\`bash
npx prisma db seed
\`\`\`
```

### 2.2 スキルの使い方

Claude Codeで以下のように呼び出します:
```
/deploy
/test --coverage
/migrate
/seed
```

## 3. フック

### 3.1 フック設定

`.claude/settings.json`:
```json
{
  "hooks": {
    "PreCommit": [
      {
        "command": "npm run lint",
        "description": "ESLint実行"
      },
      {
        "command": "npm run type-check",
        "description": "TypeScriptチェック"
      }
    ],
    "PostCommit": [
      {
        "command": "npm run test",
        "description": "テスト実行"
      }
    ]
  }
}
```

### 3.2 フックの種類

| フック | タイミング | 用途 |
|--------|-----------|------|
| PreCommit | コミット前 | Lint, Format, Type Check |
| PostCommit | コミット後 | テスト実行 |
| PrePush | プッシュ前 | 全テスト実行 |
| OnFileChange | ファイル変更時 | 自動フォーマット |

## 4. サブエージェント

### 4.1 モデル選択ガイドライン

| タスク種類 | 推奨モデル | 理由 |
|-----------|-----------|------|
| アーキテクチャ設計 | Opus | 複雑な推論が必要 |
| バグ調査 | Opus | 深い分析が必要 |
| CRUD実装 | Sonnet | 定型的なコード生成 |
| ファイル検索 | Haiku | 単純な探索タスク |
| 並列タスク | Haiku | コスト効率 |

### 4.2 サブエージェント活用例

**複雑な設計タスク（Opus）:**
```
Task(
  subagent_type="Plan",
  model="opus",
  prompt="認証システムのアーキテクチャを設計してください"
)
```

**コード探索（Haiku）:**
```
Task(
  subagent_type="Explore",
  model="haiku",
  prompt="顧客関連のAPIエンドポイントを探してください"
)
```

**並列タスク（Haiku）:**
```
# 複数のファイルを同時に検索
Task(subagent_type="Explore", model="haiku", run_in_background=true, ...)
Task(subagent_type="Explore", model="haiku", run_in_background=true, ...)
```

## 5. CLAUDE.md の活用

プロジェクトルートの`CLAUDE.md`にプロジェクト固有の情報を記載することで、Claude Codeがコンテキストを理解しやすくなります。

### 含めるべき情報
- プロジェクト概要
- 技術スタック
- ディレクトリ構造
- 開発コマンド
- コーディング規約
- 詳細ドキュメントへのリンク

## 6. 効率的な開発フロー

### 6.1 新機能開発
```
1. /plan で設計を検討（Planエージェント）
2. GitHub Issueを確認
3. feature/ブランチを作成
4. 実装（Sonnetで十分）
5. /test でテスト
6. PR作成
```

### 6.2 バグ修正
```
1. Exploreエージェントで関連コードを探索
2. Opusで原因分析
3. 修正実装
4. /test でテスト
5. PR作成
```

### 6.3 リファクタリング
```
1. Exploreエージェントで影響範囲を確認
2. Planエージェントで計画立案
3. 段階的に実装
4. /test でテスト
```

## 7. トラブルシューティング

### MCP接続エラー
```bash
# MCPサーバーの状態確認
claude mcp status

# 再接続
claude mcp restart
```

### フック実行エラー
```bash
# フックをスキップしてコミット（緊急時のみ）
git commit --no-verify -m "message"
```

### サブエージェントのタイムアウト
```
# タイムアウト時間を延長
Task(..., timeout=300000)  # 5分
```
