# システムアーキテクチャ

在庫管理システムの技術アーキテクチャの詳細説明です。

## アーキテクチャ概要

### システム構成
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (React SPA)   │◄──►│   (Express.js)  │◄──►│   (PostgreSQL)  │
│                 │    │                 │    │                 │
│  - React 18     │    │  - Express.js   │    │  - Primary DB   │
│  - TypeScript   │    │  - TypeScript   │    │  - Session Store│
│  - Vite         │    │  - Drizzle ORM  │    │  - Neon Hosted  │
│  - Shadcn UI    │    │  - WebSocket    │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## フロントエンド アーキテクチャ

### React アプリケーション構造
```
client/src/
├── components/          # 再利用可能コンポーネント
│   └── ui/             # Shadcn UIコンポーネント
├── pages/              # ページコンポーネント
│   ├── dashboard.tsx   # ダッシュボード
│   ├── inventory.tsx   # 在庫管理
│   ├── orders.tsx      # 注文管理
│   └── ...
├── hooks/              # カスタムフック
│   ├── use-auth.tsx    # 認証管理
│   └── use-toast.tsx   # 通知管理
├── lib/                # ユーティリティ
│   └── queryClient.ts  # React Query設定
└── App.tsx             # メインアプリケーション
```

### 主要技術スタック

**React エコシステム**:
- **React 18**: 最新の並行機能とSuspense対応
- **TypeScript**: 完全な型安全性
- **Wouter**: 軽量ルーティング（React Router代替）
- **TanStack React Query v5**: サーバー状態管理

**UI フレームワーク**:
- **Shadcn UI**: Radix UI + Tailwind CSSベースのコンポーネント
- **Tailwind CSS**: ユーティリティファーストのスタイリング
- **Lucide React**: モダンアイコンライブラリ
- **Framer Motion**: アニメーション

**フォーム管理**:
- **React Hook Form**: 高性能フォーム管理
- **Zod**: スキーマベースバリデーション
- **@hookform/resolvers**: React Hook Form + Zod統合

### 状態管理パターン

**サーバー状態**:
```typescript
// React Query を使用したデータフェッチ
const { data: items = [], isLoading } = useQuery<InventoryItem[]>({
  queryKey: ['/api/inventory'],
  queryFn: defaultFetcher
});
```

**クライアント状態**:
```typescript
// React useState for local state
const [searchQuery, setSearchQuery] = useState("");
const [currentPage, setCurrentPage] = useState(1);
```

**認証状態**:
```typescript
// カスタムフックによる認証状態管理
const { user, login, logout, isLoading } = useAuth();
```

## バックエンド アーキテクチャ

### Express.js アプリケーション構造
```
server/
├── routes.ts           # APIエンドポイント定義
├── storage.ts          # データアクセス層
├── db.ts              # データベース接続
├── index.ts           # サーバー設定
├── vite.ts            # Vite統合
└── services/          # ビジネスロジック
```

### 主要技術スタック

**サーバーフレームワーク**:
- **Express.js**: Webアプリケーションフレームワーク
- **TypeScript**: サーバーサイド型安全性
- **TSX**: TypeScript実行環境
- **CORS**: クロスオリジンリソース共有

**データベース層**:
- **Drizzle ORM**: 型安全SQLクエリビルダー
- **Drizzle Kit**: マイグレーション管理
- **@neondatabase/serverless**: Neon PostgreSQL接続
- **Connection Pooling**: 効率的なDB接続管理

**認証・セッション**:
- **Passport.js**: 認証ミドルウェア
- **Passport Local**: ローカル認証戦略
- **Express Session**: セッション管理
- **Connect PG Simple**: PostgreSQLセッションストア

**リアルタイム通信**:
- **WebSocket (ws)**: リアルタイム通知
- **在庫アラート**: 閾値超過時の即座通知
- **注文ステータス**: 注文状況変更の通知

### API設計パターン

**RESTful エンドポイント**:
```
GET    /api/inventory          # 在庫一覧取得
POST   /api/inventory          # 新規アイテム作成
PUT    /api/inventory/:id      # アイテム更新
DELETE /api/inventory/:id      # アイテム削除

GET    /api/orders             # 注文一覧取得
POST   /api/orders             # 注文作成
PUT    /api/orders/:id/approve # 注文承認
PUT    /api/orders/:id/reject  # 注文却下
```

**エラーハンドリング**:
```typescript
// 統一されたエラーレスポンス
interface ErrorResponse {
  error: string;
  message: string;
  statusCode: number;
}
```

## データベース アーキテクチャ

### PostgreSQL スキーマ設計

**主要テーブル関係**:
```
users (1) ──── (*) usage_history
  │
  └─── (*) orders
            │
            └─── (1) inventory ──── (1) suppliers
                      │
                      └─── (*) stock_alerts
```

**インデックス戦略**:
- **Primary Keys**: 全テーブルで自動インクリメントID
- **Foreign Keys**: 参照整合性保証
- **Search Indexes**: 名前、カテゴリー検索用
- **Composite Indexes**: 複合条件クエリ最適化

### データベース機能

**トリガー**:
```sql
-- 在庫レベル変更時の自動アラート生成
CREATE OR REPLACE FUNCTION check_inventory_threshold()
RETURNS TRIGGER AS $$
BEGIN
  -- 閾値チェック ロジック
END;
$$ LANGUAGE plpgsql;
```

**制約**:
- **CHECK制約**: データ整合性保証
- **UNIQUE制約**: 重複防止
- **NOT NULL制約**: 必須フィールド保証

## セキュリティ アーキテクチャ

### 認証フロー
```
1. User Login Request
   ├── Username/Password Validation
   ├── Passport.js Authentication
   ├── Session Creation (PostgreSQL)
   └── Role-based Access Control

2. Subsequent Requests
   ├── Session Validation
   ├── CSRF Protection
   ├── Route Authorization
   └── API Response
```

### セキュリティ対策

**入力検証**:
- **Zod Schema**: クライアント・サーバー両方での検証
- **SQL Injection**: Drizzle ORMのパラメータ化クエリ
- **XSS Protection**: 入力サニタイゼーション

**アクセス制御**:
```typescript
// ミドルウェアベースの認証
async function requireAuth(req: Request, res: Response, next: NextFunction) {
  const sessionId = req.headers.authorization?.replace('Bearer ', '');
  const user = await getSessionUser(sessionId);
  if (!user) return res.status(401).json({ message: 'Unauthorized' });
  req.user = user;
  next();
}
```

## パフォーマンス最適化

### フロントエンド最適化

**コード分割**:
```typescript
// 動的インポートによるコード分割
const LazyComponent = lazy(() => import('./components/HeavyComponent'));
```

**キャッシング戦略**:
- **React Query**: サーバーデータキャッシング
- **Stale While Revalidate**: バックグラウンド更新
- **Cache Invalidation**: データ変更時の適切な無効化

### バックエンド最適化

**データベース最適化**:
- **Connection Pooling**: 接続数制限と再利用
- **Query Optimization**: インデックス活用
- **Pagination**: 大量データの効率的な処理

**メモリ管理**:
- **Session Store**: PostgreSQLベースの永続化
- **Memory Leaks**: 適切なリソース解放
- **GC Optimization**: Node.jsガベージコレクション調整

## スケーラビリティ考慮

### 水平スケーリング
- **Stateless Architecture**: セッション外部化
- **Load Balancing**: 複数インスタンス対応
- **Database Sharding**: 将来の大規模化対応

### 垂直スケーリング
- **CPU Intensive**: 計算処理の最適化
- **Memory Intensive**: 大量データ処理対応
- **I/O Intensive**: データベースアクセス最適化

## 開発・運用ツール

### 開発環境
```json
{
  "scripts": {
    "dev": "NODE_ENV=development tsx server/index.ts",
    "build": "vite build",
    "db:push": "drizzle-kit push:pg",
    "db:studio": "drizzle-kit studio"
  }
}
```

### 監視・ログ
- **Application Logs**: 構造化ログ出力
- **Database Logs**: クエリパフォーマンス監視
- **Error Tracking**: エラー捕捉と分析
- **Health Checks**: システム稼働状況確認

### デプロイメント
- **Build Process**: Viteによる本番最適化
- **Environment Variables**: 設定の外部化
- **Database Migration**: Drizzle Kitによる自動化
- **Zero-Downtime Deploy**: ローリングデプロイ対応

---

このアーキテクチャは、保守性、スケーラビリティ、セキュリティを重視して設計されており、現代的なWeb開発のベストプラクティスに準拠しています。