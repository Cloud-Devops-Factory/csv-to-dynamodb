# DynamoDB Schema Migration Tool

## 大前提
- DynamoDBはスキーマレスなNoSQL DBであるため、通常の属性の追加はテーブル定義を変更することなく自由に可能
- ただし、テーブル定義に関わる部分として
   - 1. Partition Key = Hash Key
   - 2. Sort Key = Range Key
   - 3. LSI
   - 4. GSI
- の4つが存在する

これら4つを変更することをSchema Migrationと呼ぶこととする。

## LSI, GSIの追加
自由に追加が可能

ただし、LSIは1テーブルに対して5つまでしか作成できないという制約がある。

## LSI, GSIの削除
基本的には、不可能

どうしても削除したい場合は、DynamoDBのバックアップ/復元機能を使うのがシームレス

### バックアップ/復元機能によるLSI / GSIの削除
既存のTable AのGSI B, LSI Cを削除する場合を考える。

- 1. マネジメントコンソールからTable Aをオンデマンドバックアップ
- 2. マネジメントコンソールからTable Aを削除
- 3. 1のバックアップからTableを復元
- 4. 復元のoptionを選択する画面で「セカンダリインデックスなしで復元」を選択して復元

## Partition KeyとSort Keyの変更
基本的には不可能

どうしても変更したい場合は、新しいテーブルを作成して、古いテーブルからデータを移し替える必要がある。

### テーブルの作り直しによるHash Key / Sort Keyのmigration
既存のTable A (Partition Key=shopId、Sort Key=orderId) を
新しいTable B (Partition Key=transactionId、Sort Key = shopId)に変更する場合を考える。

- 1. DynamoDBのマネジメントコンソールからTable AをCSVエクスポート
- 2. マネジメントコンソールからTable Aを削除してTableBを作成。Cloudformationを使用している場合は既存Cloudformation stackのTable Aの定義部分を削除し、Table Bの定義を追加してDeploy
- 3. マネジメントコンソールのCloudformationの「スタックの作成」から「テンプレートファイルのアップロード」を選択
- 4. 当リポジトリの`csvToDynamo.template`をuploadし、「新たに作成するCSVアップロード用のS3 Bucket名」「DynamoDB Table Bの名前」「1でエクスポートしたCSVファイル名」を入力し、S3 BucketとそのBucketにuploadされたCSVをDynamoDBにInsertするLambdaを作成
- 5. S3 Bucketに1. のcsvファイルをupload
- 6. DynamoDBのTable Bの中身を確認
