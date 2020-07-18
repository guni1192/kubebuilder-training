# Webhookの生成

Kubernetesには、Admission Webhookと呼ばれる拡張機能があります。
これは特定のリソースを作成・更新する際にWebhook APIを呼び出し、バリデーションやリソースの書き換えをおこなうための機能です。

`kubebuilder`コマンドでは、以下の3種類のオプションで生成するWebhookを指定することができます。

- `--programmatic-validation`: リソースのバリデーションをおこなうためのWebhook
- `--defaulting`: リソースのフィールドにデフォルト値を設定するためのWebhook
- `--conversion`: カスタムリソースのバージョンアップ時にリソースの変換をおこなうためのWebhook

ここでは`--programmatic-validation`と`--defaulting`を指定して、Tenantリソース用のWebhookを生成してみましょう。

```console
$ kubebuilder create webhook --group multitenancy --version v1 --kind Tenant --programmatic-validation --defaulting
$ make manifests
```

以下のファイルが新たに追加されました。

```
├── api
│    └── v1
│        ├── tenant_webhook.go
│        └── zz_generated.deepcopy.go
├── config
│    └── webhook
│        ├── kustomization.yaml
│        ├── kustomizeconfig.yaml
│        ├── manifests.yaml
│        └── service.yaml
└── main.go
```

`api/v1/tenant_webhook.go`がWebhook実装の雛形になります。
このファイルにWebhookの実装を追加していくことになります。

`api/v1/zz_generated.deepcopy.go`は自動生成されるコードなので編集しないようにしてください。

`config/webhook`下のファイルは、Webhook機能を利用するために必要なマニフェストになります。

`main.go`には、以下のようなWebhookの初期化をおこなうためのコードが追加されています。

```go
	if err = (&multitenancyv1.Tenant{}).SetupWebhookWithManager(mgr); err != nil {
		setupLog.Error(err, "unable to create webhook", "webhook", "Tenant")
		os.Exit(1)
	}
```