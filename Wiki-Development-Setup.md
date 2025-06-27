# 開発環境セットアップ

在庫管理システムの開発環境構築手順です。

## 前提条件

### 必要ソフトウェア
- **Node.js**: v18以上推奨
- **npm**: v8以上（Node.jsに同梱）
- **Git**: バージョン管理用
- **PostgreSQL**: ローカル開発用（またはNeon使用）
- **VSCode**: 推奨エディタ（TypeScript対応）

### 推奨ブラウザ
- Chrome（最新版）
- Firefox（最新版）
- Safari（最新版）
- Edge（最新版）

## プロジェクトのクローン

```bash
# リポジトリをクローン
git clone [リポジトリURL]
cd inventory-management-system

# 依存関係をインストール
npm install
```

## 環境変数の設定

### 必要な環境変数
`.env`ファイルを作成し、以下の変数を設定：

```env
# データベース接続
DATABASE_URL=postgresql://username:password@localhost:5432/inventory_db

# PostgreSQL接続詳細
PGHOST=localhost
PGPORT=5432
PGUSER=postgres
PGPASSWORD=your_password
PGDATABASE=inventory_db

# セッション設定
SESSION_SECRET=your-secret-key-here

# 開発環境
NODE_ENV=development
```

### Neonデータベースを使用する場合
```env
DATABASE_URL=postgresql://[username]:[password]@[endpoint]/[dbname]?sslmode=require
```

## データベースセットアップ

### ローカルPostgreSQLの場合

```bash
# PostgreSQL接続
psql -U postgres

# データベース作成
CREATE DATABASE inventory_db;

# ユーザー作成（必要に応じて）
CREATE USER inventory_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE inventory_db TO inventory_user;
```

### スキーマの適用

```bash
# データベーススキーマをプッシュ
npm run db:push

# 初期データの投入（必要に応じて）
npm run db:seed
```

## 開発サーバーの起動

```bash
# 開発サーバー起動（フロントエンド + バックエンド）
npm run dev
```

起動後、以下でアクセス可能：
- **フロントエンド**: http://localhost:5000
- **バックエンド API**: http://localhost:5000/api

## 開発ツール

### Drizzle Studio（データベース管理）
```bash
# Drizzle Studioを起動
npm run db:studio
```
ブラウザで https://local.drizzle.studio が開き、データベースを視覚的に管理できます。

### TypeScript型チェック
```bash
# 型チェック実行
npx tsc --noEmit

# ウォッチモード
npx tsc --noEmit --watch
```

### ESLint（コード品質）
```bash
# ESLint実行
npm run lint

# 自動修正
npm run lint:fix
```

## プロジェクト構造理解

### ディレクトリ構成
```
inventory-management-system/
├── client/                 # フロントエンドコード
│   └── src/
│       ├── components/     # 再利用可能コンポーネント
│       ├── pages/          # ページコンポーネント
│       ├── hooks/          # カスタムフック
│       └── lib/            # ユーティリティ
├── server/                 # バックエンドコード
│   ├── routes.ts           # API定義
│   ├── storage.ts          # データアクセス
│   ├── db.ts               # DB接続
│   └── index.ts            # サーバー設定
├── shared/                 # 共通型定義
│   └── schema.ts           # Drizzleスキーマ
├── package.json            # 依存関係
├── vite.config.ts          # Vite設定
├── tailwind.config.ts      # Tailwind設定
└── drizzle.config.ts       # Drizzle設定
```

### 重要ファイル

**設定ファイル**:
- `vite.config.ts`: ビルド設定とプロキシ
- `tailwind.config.ts`: スタイル設定
- `drizzle.config.ts`: データベース設定
- `tsconfig.json`: TypeScript設定

**エントリーポイント**:
- `client/src/App.tsx`: フロントエンドのメイン
- `server/index.ts`: バックエンドのメイン
- `shared/schema.ts`: データベーススキーマ

## 開発ワークフロー

### 基本的な開発手順

1. **機能ブランチの作成**
```bash
git checkout -b feature/new-feature
```

2. **データベーススキーマの変更**
```bash
# schema.tsを編集後
npm run db:push
```

3. **開発・テスト**
```bash
# 開発サーバー起動
npm run dev

# 別ターミナルで型チェック
npx tsc --noEmit --watch
```

4. **コミット・プッシュ**
```bash
git add .
git commit -m "feat: add new feature"
git push origin feature/new-feature
```

### データベース変更の手順

1. **スキーマ更新**: `shared/schema.ts`を編集
2. **プッシュ**: `npm run db:push`で反映
3. **確認**: Drizzle Studioで変更確認
4. **テストデータ**: 必要に応じて初期データを追加

### 新しいページ追加の手順

1. **ページコンポーネント作成**: `client/src/pages/new-page.tsx`
2. **ルート追加**: `client/src/App.tsx`にルート追加
3. **ナビゲーション更新**: メニューにリンク追加
4. **API エンドポイント**: 必要に応じて`server/routes.ts`に追加

## デバッグ技法

### フロントエンドデバッグ

**React Developer Tools**:
- ブラウザ拡張機能をインストール
- コンポーネント階層の確認
- state/propsの監視

**ブラウザ開発者ツール**:
```javascript
// Console logging
console.log('Debug info:', data);

// Network tab
// APIリクエスト/レスポンスの確認

// React Query Devtools
// クエリの状態確認
```

### バックエンドデバッグ

**サーバーログ**:
```typescript
console.log('API called:', req.path);
console.log('User:', req.user);
console.log('Body:', req.body);
```

**データベースクエリ確認**:
```bash
# Drizzle Studio でクエリ実行
npm run db:studio
```

### よくあるエラーと対処法

**ポートが使用中**:
```bash
# プロセス確認
lsof -i :5000

# プロセス終了
kill -9 [PID]
```

**データベース接続エラー**:
1. 環境変数の確認
2. PostgreSQLサービスの起動確認
3. 接続情報の再確認

**TypeScript エラー**:
1. 型定義の確認
2. インポートパスの確認
3. `npm install`で依存関係の再インストール

## 本番環境へのデプロイ

### ビルド手順
```bash
# 本番ビルド
npm run build

# ビルド結果確認
ls -la dist/
```

### 環境変数（本番）
```env
NODE_ENV=production
DATABASE_URL=[本番データベースURL]
SESSION_SECRET=[強力なシークレットキー]
```

### デプロイ前チェックリスト
- [ ] TypeScript エラーなし
- [ ] ESLint エラーなし
- [ ] データベースマイグレーション実行
- [ ] 環境変数設定完了
- [ ] セキュリティ設定確認

## トラブルシューティング

### パフォーマンス問題
- React Query キャッシュの確認
- データベースクエリの最適化
- バンドルサイズの分析

### セキュリティチェック
- 依存関係の脆弱性スキャン: `npm audit`
- 環境変数の適切な設定
- CORS設定の確認

### 互換性問題
- Node.jsバージョンの確認
- ブラウザサポートの検証
- TypeScriptバージョンの互換性

---

**サポート**: 開発中に問題が発生した場合は、GitHubのIssueまたはチーム内のコミュニケーションツールで報告してください。