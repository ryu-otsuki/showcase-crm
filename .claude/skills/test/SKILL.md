# /test - テスト実行

プロジェクトのテストを実行します。

## 実行内容

Vitestを使用してユニットテスト・統合テストを実行します。

## 使用方法

```
/test              # 全テスト実行
/test --coverage   # カバレッジレポート付き
/test --watch      # ウォッチモード
/test [file]       # 特定ファイルのみ
```

## 実行コマンド

```bash
# 基本
npm run test

# カバレッジ付き
npm run test -- --coverage

# ウォッチモード
npm run test -- --watch

# 特定ファイル
npm run test -- [file]
```

## テスト構成

```
tests/
├── unit/           # ユニットテスト
│   ├── utils/
│   └── hooks/
├── integration/    # 統合テスト
│   └── api/
└── e2e/            # E2Eテスト（Playwright）
```

## カバレッジ目標

- 全体: 80%以上
- 重要なビジネスロジック: 90%以上
