````markdown
README — Trend Micro Vision One プラグイン（Microsoft Security Copilot）

概要

このフォルダーには、Security Copilot プラグインのマニフェスト `trendmicro-plugin.yaml`（YAML）と、Trend Micro Vision One（Workbench アラートおよびエンドポイント セキュリティ）に対する一連のスキルを公開する OpenAPI 仕様 `trendmicro-openapi.yaml` が含まれています。

この README では、プラグインのアップロードと設定方法、Security Copilot からのテスト方法、接続を検証するための curl / PowerShell の例を示します。

重要なセキュリティ注意事項

- 本番環境で長期間有効なベアラートークンやシークレットをマニフェストに保存しないでください。Security Copilot プラグインの「セットアップ」UI を使用してトークンを入力するか、シークレットマネージャーを統合してください。
- 不要になったトークンはローテーションおよび取り消しを行ってください。

前提条件

- カスタムプラグインをアップロードできる権限を持つ Microsoft Security Copilot へのアクセス。
- Trend Micro Vision One / XDR の API キー。公式ドキュメント:
  -- https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-platform-api-keys
  -- https://automation.trendmicro.com/xdr/api-v3/
- （任意）Security Copilot がベンダー提供の OpenAPI 仕様を取得できない場合は、OpenAPI 仕様を到達可能な URL にホストし、プラグインマニフェストの `OpenApiSpecUrl` フィールドを更新してください（例: `trendmicro-openapi.yaml`）。
  サンプルプラグインでは、このリポジトリ内に保存されている OpenAPI を使用できます。サンプルマニフェストで既に設定されています：
  OpenApiSpecUrl: https://raw.githubusercontent.com/doug-msft/SecurityCopilot/main/TrendMicro/trendmicro-openapi.yaml

アップロードとセットアップ手順（Security Copilot）

1. Security Copilot（https://securitycopilot.microsoft.com/）を開き、プラグインを管理できるアカウントでサインインします。
2. 「Manage plugins」→「Custom」→「Add plugin」に移動します。
3. YAML ファイル形式を選択し、`trendmicro-plugin.yaml` をアップロードします。
4. アップロード後、プラグインをクリックして「Set up」を選択します。API キーの入力を求められたら次を指定してください。
   - "API Key Value": ベアラートークンまたは API キーを貼り付けます（マニフェストに埋め込むより UI に入力するほうが安全です）
5. プラグイン設定を保存します。

Security Copilot でのプラグインのテスト

設定後は、次のような自然言語プロンプトを使用できます。
- 「過去 24 時間の Trend Micro アラートを一覧表示して」
- 「アラート <alertId> の詳細を表示して」
- 「ホスト名が 'webserver01' のエンドポイントを一覧表示して」
- 「エンドポイントの詳細を取得して」

直接 API テスト（curl / PowerShell）

事前にベアラートークンがあることを前提とした例です。

curl の例（<TOKEN> を置き換えてください）:

```
curl -H "Authorization: Bearer <TOKEN>" \
  "https://api.xdr.trendmicro.com/v3.0/workbench/alerts?limit=10"
```

PowerShell の例（<TOKEN> を置き換えてください）:

```powershell
$token = '<TOKEN>'
Invoke-RestMethod -Method Get -Uri 'https://api.xdr.trendmicro.com/v3.0/workbench/alerts?limit=10' -Headers @{ Authorization = "Bearer $token" }
```

テナントが異なるベース URL を使用している場合やトークン交換エンドポイントが必要な場合は、Trend Micro XDR のドキュメントを参照し、URL を適宜調整してください。

トラブルシューティング

- 401 Unauthorized: トークンの有効性、期限切れ、ヘッダー名（Authorization と x-api-key）を確認してください。
- 403 Forbidden: Trend Micro 管理コンソールで API 権限を確認してください。
- 404 Not Found: API パスが Trend Micro XDR のバージョンに一致しているか確認してください（/v3 または /v3.0 の差異など）。OpenApiSpecUrl を更新してください。サンプル OpenAPI は `https://api.xdr.trendmicro.com` をベース URL としています。

お問い合わせ / 付記

必要であれば、環境変数からベアラートークンを読み取り、スキルで使われるリクエストを実行する簡単な PowerShell スクリプトとテストハーネスを追加できます。

````
