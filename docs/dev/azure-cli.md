# Azure CLI のよく使うコマンドとフロー

Azure CLI を使用する際によく利用するコマンドと、一般的な操作フローについて記述します。

## 1. 認証と環境設定

*   `az login`: Azure アカウントにログインします。ブラウザが起動し、認証を求められます。
*   `az account set --subscription "Your Subscription Name or ID"`: 複数のサブスクリプションがある場合、使用するサブスクリプションを設定します。
*   `az configure --defaults group=YourResourceGroup location=japaneast`: デフォルトのリソースグループとロケーションを設定すると、コマンド実行時に毎回指定する必要がなくなります。

## 2. リソースグループの操作

*   `az group create --name MyResourceGroup --location japaneast`: 新しいリソースグループを作成します。
*   `az group list --output table`: 既存のリソースグループを一覧表示します。
*   `az group show --name MyResourceGroup`: 特定のリソースグループの詳細を表示します。
*   `az group delete --name MyResourceGroup --yes --no-wait`: リソースグループとその中のすべてのリソースを削除します。`--yes` は確認なしで削除、`--no-wait` は削除処理を待たずに次のコマンドを実行します。

## 3. 仮想マシンの操作

*   `az vm create --resource-group MyResourceGroup --name MyVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys`: Ubuntu LTS イメージで仮想マシンを作成します。SSH キーも自動生成されます。
*   `az vm list --output table`: 仮想マシンの一覧を表示します。
*   `az vm show --resource-group MyResourceGroup --name MyVM --query "publicIps" --output tsv`: 特定の仮想マシンのパブリック IP アドレスを表示します。
*   `az vm start --resource-group MyResourceGroup --name MyVM`: 仮想マシンを起動します。
*   `az vm stop --resource-group MyResourceGroup --name MyVM`: 仮想マシンを停止します。
*   `az vm delete --resource-group MyResourceGroup --name MyVM --yes`: 仮想マシンを削除します。

## 4. ストレージアカウントの操作

*   `az storage account create --name mystorageaccountname --resource-group MyResourceGroup --location japaneast --sku Standard_LRS`: ストレージアカウントを作成します。
*   `az storage account list --output table`: ストレージアカウントの一覧を表示します。
*   `az storage account show-connection-string --name mystorageaccountname --resource-group MyResourceGroup --query "connectionString" --output tsv`: ストレージアカウントの接続文字列を表示します。
*   `az storage container create --name mycontainer --account-name mystorageaccountname`: ストレージコンテナを作成します。
*   `az storage blob upload --container-name mycontainer --file "path/to/local/file.txt" --name "remote-file.txt" --account-name mystorageaccountname`: ファイルを Blob ストレージにアップロードします。

## 5. Webアプリの操作

*   `az appservice plan create --name MyAppServicePlan --resource-group MyResourceGroup --location japaneast --sku F1`: App Service プランを作成します。
*   `az webapp create --resource-group MyResourceGroup --plan MyAppServicePlan --name MyWebApp`: Web アプリを作成します。
*   `az webapp list --output table`: Web アプリの一覧を表示します。
*   `az webapp browse --name MyWebApp`: Web アプリをブラウザで開きます。

## 6. サービスプリンシパルの操作 (GitHub Actions などでの利用)

`az ad sp create-for-rbac` コマンドは、Azure Active Directory (AD) にサービスプリンシパルを作成し、特定のロールとスコープを割り当てるために使用します。これは、GitHub Actions などの CI/CD ツールから Azure リソースにアクセスする際に必要となる認証情報（サービスプリンシパル）を生成するために特に有用です。

### コマンドの書式

```bash
az ad sp create-for-rbac --name "YourServicePrincipalName" \
  --role "Contributor" \
  --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group-name-1} \
  --sdk-auth
```

### パラメータの説明

*   `--name "YourServicePrincipalName"`: 作成するサービスプリンシパルの名前を指定します。分かりやすい名前をつけましょう（例: `"github-actions-survice-name"`）。
*   `--role "Contributor"`: サービスプリンシパルに割り当てるロールを指定します。通常は `"Contributor"` (共同作成者) を指定しますが、必要に応じてより制限されたロール (`"Reader"` など) を指定することも可能です。
*   `--scopes`: サービスプリンシパルがアクセスできる範囲（スコープ）を指定します。複数のリソースグループやサブスクリプションに対してアクセス権を付与したい場合は、スペース区切りで複数指定できます。
    *   `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name-1}`: サービスプリンシパルがアクセスを許可されるリソースグループのIDを指定します。`{subscription-id}` は対象のサブスクリプションID、`{resource-group-name-1}` は対象のリソースグループ名に置き換えてください。
    *   `/subscriptions/{subscription-id}`: サブスクリプション全体にアクセス権を付与することも可能です。
*   `--sdk-auth`: このオプションを指定すると、出力が JSON 形式で返され、Azure SDKs (SDK-auth 形式) で直接使用できる認証情報が含まれます。これは特に GitHub Actions のワークフローなどで `AZURE_CREDENTIALS` 環境変数として設定する際に便利です。

### 使用例と出力

```bash
az ad sp create-for-rbac --name "github-actions-survice-name" \
  --role contributor \
  --scopes /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group-1 \
  --sdk-auth
```

上記のコマンドを実行すると、以下のような JSON 出力が得られます。

```json
{
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

この JSON 出力全体を、GitHub Actions のシークレット (`AZURE_CREDENTIALS` など) として安全に保存し、ワークフローから利用することで、Azure リソースへの認証とデプロイが可能になります。
