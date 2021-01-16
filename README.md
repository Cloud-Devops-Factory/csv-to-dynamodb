# DynamoDB Schema Migration Tool

## 大前提
- DynamoDBはスキーマレスなNoSQL DBであるため、通常の属性の追加はテーブル定義を変更することなく自由に可能
- ただし、テーブル定義に関わる部分として
   - 1. Partition Key = Hash Key
   - 2. Sort Key = Range Key
   - 3. LSI
   - 4. GSI
- の4つが存在する

既存のテーブルに対して上記の4項目を変更することをSchema Migrationと呼ぶこととする。

## GSIの追加/削除
既存テーブルに対しても自由に追加/削除が可能

## LSIの追加/削除
既存テーブルに対しての追加/削除は不可能

LSIはテーブル作成時にのみ作成される。

どうしても追加/削除したい場合はテーブルを作り直すことになる。

参照: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html#LSI.Creating (`Local secondary indexes on a table are created when the table is created.`)

## Partition KeyとSort Keyの変更
既存テーブルに対しての変更は不可能

どうしても変更したい場合は、テーブルを作り直すことになる。

## テーブルの作り直しに伴うデータ移行
既存テーブルに対するLSIの追加/削除、Partition KeyとSort Keyの変更を行う際にはテーブルを作り直す必要がある。
万が一、運用中のテーブルに対して作り直しを行う必要が発生した場合は以下の手順で行う。

- 1. DynamoDBのマネジメントコンソールから旧Tableの中身をCSVエクスポート
- 2. マネジメントコンソールから旧Tableを削除して新Tableを作成
    - Cloudformationを使用している場合は既存Cloudformation Stackの旧Tableの定義部分を削除してDeploy
    - 新Tableの定義を追加してDeploy
- 3. マネジメントコンソールのCloudformationの「スタックの作成」から「テンプレートファイルのアップロード」を選択
- 4. 当リポジトリの`cloudformation/csvToDynamo.template`をuploadし、「新たに作成するCSVアップロード用のS3 Bucket名」「DynamoDB 新Tableの名前」「1でエクスポートしたCSVファイル名」を入力し、S3 BucketとそのBucketにuploadされたCSVをDynamoDBにInsertするLambdaを作成
- 5. S3 Bucketに1. のcsvファイルをupload
- 6. DynamoDBの新Tableの中身を確認
