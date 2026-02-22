# Product Requirement Document: メルカリ自動出品システム (PoC)

## 1. システム概要
### 1.1 目的
- メルカリへの出品作業を自動化し、手動運用の工数を削減する。
- 決められたスケジュールに基づき、一貫した出品・再出品を実現する。

### 1.2 ターゲット
- ローカル環境（Windows/Mac）で動作する自動実行スクリプト。
- 将来的にクラウド（AWS/Docker環境）への移行を視野に入れた設計。

## 2. 業務フロー


1. **スケジュール起動**: 毎日18時、22時（日本時間）にプロセスが起動。
2. **環境チェック**: 接続元IPを判定。日本国外の場合は即座に処理を中断。
3. **データ取得**: DBから当日の曜日・時間に対応する出品商品リストを取得。
4. **自動出品**: Playwright等のブラウザ制御ライブラリを用いて出品を実行。
5. **完了・ログ記録**: 出品結果、またはエラー内容をDB/ログファイルに記録。

## 3. 主要機能 (PoC範囲)
### 3.1 認証・ログイン
- **初回ログイン**: 手動ログインを許容。
- **セッション維持**: ログイン後のCookieやLocalStorageを永続化し、2回目以降はログイン操作をスキップ。
### 3.2 出品画面までの画面遷移
#### 未ログイン
1.top
https://jp.mercari.com/
1.1 ログインボタン押下

2.login
https://login.jp.mercari.com/signin?acr_values=urn%3Amercari%3Aacr%3Asf&nonce=P7jU06uBeDUJqpCZqIoY4L5xrdne1Us97zbtsNOKpzuyamCE3xtDNlNwcI8z&params=client_id%3DbP4zN6jIZQeutikiUFpbx307DVK1pmoW%26code_challenge%3DDkmnZULGpsOEKeBPzRc9whwe1X44U1A7u0DX3rLggQg%26code_challenge_method%3DS256%26flow_session_id%3D404820b5-456a-4b0c-b128-1507fdb40d46%26nonce%3D~CVAQooiC-LN%26onboarding_type%3Dsignin%26redirect_uri%3Dhttps%253A%252F%252Fjp.mercari.com%252Fauth%252Fcallback%26response_type%3Dcode%26scope%3Dmercari%2Bopenid%26state%3DeyJwYXRoIjoiLyIsInJhbmRvbSI6InZGaVVidlhnYUJOeCJ9%26ui_locales%3Dja
2.1 メールアドレス入力
2.2 次へボタン押下
2.3 パスキーでログイン

3.top
https://jp.mercari.com/
出品ボタン押下

#### ログイン済
1.top
https://jp.mercari.com/
出品ボタン押下

### 3.2 出品エンジン
- **ブラウザ操作**: ヘッドレスブラウザによる商品情報の自動入力。
- **画像処理**: 最大10枚の画像を順番にアップロード。
- **動的選択項目**: カテゴリ、配送設定、地域等のセレクトボックス/ラジオボタンの自動選択。

### 3.3 自動削除機能
- 出品からN日（設定値）経過した商品を特定し、自動で削除を行う。

### 3.4 安全・BAN回避策 (最重要)
- **人間らしい振る舞い**: 
    - 入力やクリックの間にランダムな待機時間（2〜5秒等）を挿入。
    - マウス移動やスクロールをシミュレート。
- **実行環境秘匿**: `stealth` プラグインを使用し、自動操作の指紋（fingerprint）を隠蔽。
- **リージョン制限**: 日本国内からの接続が確認できない場合、一切の操作を禁止。

## 4. 商品データ項目 (DB設計の基礎)
| 項目名 | 形式 | 備考 |
| :--- | :--- | :--- |
| 画像 | パス/URL (10) | 複数枚対応 |
| 商品名 | 文字列 | 最大40文字 |
| カテゴリ | 各階層ID | 要調査・マッピング |
| 商品の状態 | ID | 要調査 |
| 商品の説明 | テキスト | 改行保持 |
| 配送料の負担 | ID | 出品者/購入者 |
| 配送の方法 | ID | |
| 配送元の地域 | ID | |
| 配送までの日数 | ID | |
| 販売価格 | 数値 | |
| 販売タイプ | ID | |

## 5. 技術スタック案
- **Runtime**: Node.js (TypeScript)
- **Automation**: Playwright (マルチブラウザ対応、高い安定性)
- **Database**: SQLite (PoC用の軽量構成)
- **Architecture**: 
    - **Scraper Layer**: HTML構造に依存する部分（変更に弱いため分離）
    - **Logic Layer**: スケジュール、DB連携、出品ロジック
    - **Infra Layer**: ローカル実行用、将来のクラウドデプロイ用

## 6. 今後の課題・調査事項
- メルカリHTML構造の解析（各画面のセレクタ特定）。
- カテゴリIDおよび各選択項目の内部値（Value）の調査。
- クラウド移行時のChromiumバイナリの実行サイズとメモリ消費の検討。
- 必ずログイン済みで認証情報付けて画面表示する方法を検討。