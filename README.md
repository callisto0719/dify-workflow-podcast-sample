# Dify Podcast Workflow Sample

Difyを使用してGoogle NewsのRSSフィードからテック系ポッドキャスト記事を自動生成し、音声合成してGitHubに投稿するワークフローのサンプルです。

## 注意
1. 環境変数を変更しているため、local環境のdockerで動いているDifyでのみ動作を確認しています。
2. 試作段階のため、以下の問題を含みます。本番環境では利用しないようお願いいたします。
    - Pythonコード上でのリクエスト時に証明書を確認していない。(中間者攻撃の可能性がある)
    - Github Pagesで動かすRSS用のリポジトリのブランチ名をmaster前提でコーディングしている。

## 機能概要

このワークフローは以下の処理を自動実行します：

1. **ニュース収集**: Google NewsのRSSフィードからテクノロジー関連のニュースを取得
2. **記事生成**: LLMを使用してポッドキャスト用の読み上げ記事を作成
3. **音声合成**: OpenAI TTSを使用してテキストを音声ファイルに変換
4. **タイトル・説明文生成**: ポッドキャスト用のタイトルと説明文を自動生成
5. **GitHub投稿**: 生成した音声ファイルとメタデータをGitHubリポジトリに自動投稿

## 依存関係

### 必須プロバイダー
- **OpenAI**: LLMとTTSで使用
  - モデル: `gpt-4.1` (記事生成・タイトル生成)
  - TTS: `gpt-4o-mini-tts` (音声合成)

### 必須プラグイン
1. **langgenius/openai:0.2.3** - OpenAI APIとの連携
2. **bowenliang123/base64_codec:0.3.0** - Base64エンコーディング

### 環境変数
以下の環境変数の設定が必要です：

| 変数名 | 説明 | タイプ |
|--------|------|--------|
| `GITHUB_OWNER` | GitHubアカウント名 | string |
| `GITHUB_REPO` | GitHubリポジトリ名 | string |
| `GITHUB_PAT` | GitHub Personal Access Token | secret |

OWNER, REPOに関してはsecretで設定していないため、元の値から書き換えること。

## セットアップ手順

1. **Difyプロジェクトの準備**
   - Difyの環境を準備
   - OpenAIプロバイダーを設定

2. **GitHubの準備**
   - ポッドキャスト用のGitHubリポジトリを作成
   - Personal Access Tokenを生成（Contents権限が必要）

3. **環境変数の設定**
   - Difyの環境変数設定で上記3つの変数を設定

4. **ワークフローのインポート**
   - `podcast-workflow.yml`をDifyにインポート

## 重要な注意事項

### OpenAIレスポンス制限について
OpenAIからの大きなレスポンスを受け取れない場合があります。その場合はDifyのenvから以下の環境変数を適した値に変更してください：

HTTP_REQUEST_NODE_MAX_TEXT_SIZE

参考: https://qiita.com/lamvn/items/315b0988069019cd0140

### 音声合成について
- デフォルトではOpenAI TTSを使用
- VoiceVoxなどの代替案も検討可能（実装要検討）
- 参考: https://zenn.dev/isi00141/articles/54c5c176f6ac1a

### GitHubアップロード形式
- 音声ファイル: `/audio/{日付}-episode.wav`
- メタデータファイル: `/_posts/{日付}-{タイトル}.md`

## 想定される使用方法

1. 日次実行でGoogle Newsから最新のテクニュースを収集
2. LLMが5分程度で読み上げ可能な記事にまとめ
3. 音声合成してポッドキャスト用音声ファイルを生成
4. GitHubリポジトリに自動投稿

## トラブルシューティング

- **環境変数が設定されていない場合**: ワークフローが途中で停止します
- **GitHub PAT権限不足**: Contents権限が必要です
- **OpenAI API制限**: レート制限に注意してください
- **ファイルサイズ制限**: 大きな音声ファイルの場合はGitHub制限に注意