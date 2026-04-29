---
marp: true
theme: default
paginate: true
---

# Week 1 実行ステップ
## Spring Bootアプリ立ち上げまでの具体手順

- 対象: 小さな保険契約管理アプリ
- 目的: Week 1の到達目標を順番に実行できる状態にする
- 範囲: Spring Boot起動、DB接続、`customers` 設計、契約者登録API、詳細取得の骨組み

---

# Week 1 の到達目標

- Spring Bootアプリがローカル起動できる
- DBに接続できる
- `customers` テーブル設計が固まっている
- `POST /api/customers` が動く
- `GET /api/customers/{customerId}` または検索の骨組みがある

---

# 進め方の全体像

| Step | 内容 | ゴール |
| --- | --- | --- |
| 1 | 環境準備 | ローカル起動できる |
| 2 | DB準備 | PostgreSQLに接続できる |
| 3 | テーブル設計 | `customers` を確定する |
| 4 | 登録API実装 | 契約者登録が動く |
| 5 | 取得API実装 | 詳細取得の骨組みを作る |

---

# Step 1
## 環境を確認する

1. Javaのバージョンを確認する
2. Mavenのバージョンを確認する
3. DockerとDocker Composeを確認する
4. IDEを準備する

```powershell
java -version
mvn -version
docker --version
docker compose version
```

---

# Step 1
## 環境確認の結果

| 項目 | 確認結果 |
| --- | --- |
| Java | `java version "25" 2025-09-16 LTS` |
| Maven | `Apache Maven 3.9.11` |
| Docker | `Docker version 28.5.1` |
| Docker Compose | `v2.40.0-desktop.1` |
| OS | `Windows 11 / amd64` |

補足:
- Maven実行時のJavaも `Java version: 25` で一致
- 文字コードは `UTF-8`

---

# Step 1
## 久しぶりに触る前提での進め方

- いきなり業務APIを作らず、まず最小のSpring Bootアプリを通す
- 最初に `build` と `run` を成功させて感覚を戻す
- そのあと `Hello API` を作って MVC の流れを思い出す
- DB接続はその次に行い、最後に `customers` 実装へ入る

参考にした考え方:
- 2025年8月11日公開のZenn記事は、公式チュートリアルを先に一通り進めてから深掘りする流れだった

---

# Step 1
## Javaバージョンの方針

- 現在のローカル環境は `Java 25 LTS`
- ただし、学習用途では `Java 21 LTS` を使うと情報差分が少なく進めやすい
- 今回はまず `Java 25` のまま始めてもよいが、問題が出たら `Java 21` へ切り替える

補足:
- Spring Boot 3.2.5 の公式ドキュメントでは Java 17 以上、Java 22 までの互換記載がある
- Spring Boot 3.5.0 のミラー文書では Java 24 までの互換記載が見つかった
- そのため、`Java 25` は環境によって差分が出る可能性があると見ておく

---

# Step 1
## Spring Bootプロジェクトを作る

- 依存関係を選ぶ
- 初期起動できる最小構成を作る
- まずはDBなしで起動確認する
- 生成方法は `Spring Initializr + Maven` を基本にする

| 依存関係 | 用途 |
| --- | --- |
| Spring Web | REST API |
| Validation | 入力チェック |
| Spring Data JPA | DBアクセス |
| PostgreSQL Driver | PostgreSQL接続 |
| Lombok | ボイラープレート削減 |
| Spring Boot DevTools | 開発補助 |
| Spring Boot Test | テスト |

---

# Step 1
## プロジェクト作成時の設定

| 項目 | 推奨値 |
| --- | --- |
| Project | Maven |
| Language | Java |
| Spring Boot | 3.5系 |
| Group | `com.example` または自分のドメイン |
| Artifact | `insurance-app` |
| Packaging | Jar |
| Java | まずは 21、手元で試すなら 25 も可 |

補足:
- 久しぶりなら Gradle より Maven のほうが追いやすい
- まずは生成された形を崩さずに進める

---

# Step 1
## パッケージ構成を作る

- `controller`
- `service`
- `repository`
- `entity`
- `dto`
- `exception`
- `common`
- `config`

補足:
- 今の段階で構成を分けておくと、後で `Policy` 機能を足しやすい

---

# Step 1
## アプリの初回起動を確認する

1. `Application.java` を作る
2. まずはプレーンな状態で起動する
3. 起動ログにエラーがないことを確認する

```powershell
mvn spring-boot:run
```

- 完了条件: Spring Bootアプリがローカルで起動する

---

# Step 1
## 最小のHello APIを先に作る

- `@RestController` を1つだけ作る
- `GET /hello` で固定文字列を返す
- ここで Controller の流れを思い出す

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot";
    }
}
```

---

# Step 1
## Hello APIで確認すること

1. `mvn spring-boot:run` で起動する
2. ブラウザや `curl` で `GET /hello` を呼ぶ
3. 文字列が返ることを確認する
4. 一度 `mvn test` か `mvn package` も通しておく

```powershell
curl http://localhost:8080/hello
mvn test
mvn package
```

- 完了条件: `build` と `run` を一度通して感覚を戻せている

---

# Step 2
## PostgreSQLをDockerで起動する

1. `docker-compose.yml` を作る
2. `db` サービスだけ定義する
3. DB名、ユーザー、パスワードを決める

| 項目 | 値の例 |
| --- | --- |
| Image | `postgres:16` |
| Database | `insurance_db` |
| User | `insurance_user` |
| Password | `insurance_pass` |

---

# Step 2
## DBを起動する

```powershell
docker compose up -d
```

確認ポイント:
- コンテナが起動している
- `5432` ポートで待ち受けている
- 再起動しても同じ設定で上がる

- 完了条件: PostgreSQLコンテナが起動する

---

# Step 2
## Spring BootからDBへ接続する

`application.yml` に接続設定を入れる

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/insurance_db
    username: insurance_user
    password: insurance_pass
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
```

- 起動して接続エラーが消えればOK

---

# Step 3
## customersテーブルを設計する

最初に確定する項目はこの形で十分です。

| カラム | 型 | 制約 | 用途 |
| --- | --- | --- | --- |
| `id` | `BIGSERIAL` | PK | 内部ID |
| `customer_id` | `VARCHAR(20)` | NOT NULL, UNIQUE | 業務用ID |
| `name` | `VARCHAR(100)` | NOT NULL | 氏名 |
| `birth_date` | `DATE` | NOT NULL | 生年月日 |
| `address` | `VARCHAR(255)` | NOT NULL | 住所 |
| `phone_number` | `VARCHAR(20)` | NOT NULL | 電話番号 |
| `email` | `VARCHAR(255)` | NULL | メールアドレス |
| `created_at` | `TIMESTAMP` | NOT NULL | 作成日時 |
| `updated_at` | `TIMESTAMP` | NOT NULL | 更新日時 |

---

# Step 3
## DDLを作る

`schema.sql` に最低限このDDLを置く

```sql
CREATE TABLE customers (
  id BIGSERIAL PRIMARY KEY,
  customer_id VARCHAR(20) NOT NULL UNIQUE,
  name VARCHAR(100) NOT NULL,
  birth_date DATE NOT NULL,
  address VARCHAR(255) NOT NULL,
  phone_number VARCHAR(20) NOT NULL,
  email VARCHAR(255),
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

- 完了条件: `customers` テーブル定義が固まる

---

# Step 3
## EntityとRepositoryを作る

1. `Customer` Entityを作る
2. `CustomerRepository` を作る
3. `findByCustomerId` を用意する
4. 起動時にスキーマ整合を確認する

```java
Optional<Customer> findByCustomerId(String customerId);
```

- 完了条件: EntityとRepositoryの土台ができる

---

# Step 4
## 契約者登録APIの仕様を決める

API:
- `POST /api/customers`

リクエスト項目:
- `name`
- `birthDate`
- `address`
- `phoneNumber`
- `email`

レスポンス項目:
- `customerId`
- `name`
- `birthDate`
- `address`
- `phoneNumber`
- `email`
- `createdAt`

---

# Step 4
## リクエストバリデーションを入れる

| 項目 | ルール |
| --- | --- |
| `name` | 必須 |
| `birthDate` | 必須、未来日不可 |
| `address` | 必須 |
| `phoneNumber` | 必須、数字とハイフンのみ |
| `email` | 任意、入力時はメール形式 |

補足:
- Bean Validationで入力チェック
- 業務ルールはServiceでも確認する

---

# Step 4
## 契約者ID採番を実装する

- 形式は `C000001`
- Week 1は簡易実装でOK
- まずは学習用として `件数 + 1` 方式でも進められる

処理イメージ:
1. 現在の件数を取得
2. 次の番号を計算
3. `C` + 6桁ゼロ埋めでID生成

- 完了条件: 登録時に `customerId` が自動発行される

---

# Step 4
## ServiceとControllerを作る

Serviceの役割:
- 入力内容を受け取る
- 生年月日を確認する
- `customerId` を採番する
- Entityへ変換して保存する
- Response DTOに変換する

Controllerの役割:
- `@PostMapping("/api/customers")`
- `@Valid` で受け取る
- `201 Created` を返す

---

# Step 4
## 登録APIを動作確認する

リクエスト例:

```json
{
  "name": "山田太郎",
  "birthDate": "1995-04-01",
  "address": "東京都千代田区1-1-1",
  "phoneNumber": "090-1234-5678",
  "email": "taro.yamada@example.com"
}
```

確認ポイント:
- 登録成功する
- DBに1件入る
- `customerId` が返る

---

# Step 5
## 取得APIの骨組みを作る

おすすめは `詳細取得` を先に作ることです。

API:
- `GET /api/customers/{customerId}`

理由:
- 検索APIよりシンプル
- `findByCustomerId` をそのまま使える
- 登録APIの確認にもつながる

---

# Step 5
## 詳細取得の実装内容

1. `customerId` で検索する
2. 見つからなければ `NotFound` 例外にする
3. Response DTOへ変換する
4. Controllerで返す

完了条件:
- `GET /api/customers/C000001` で取得できる

---

# Step 5
## 余力があれば検索の骨組みも作る

API:
- `GET /api/customers?name=山田`

最初はこれで十分:
- `name` の部分一致のみ対応
- 一覧DTOを返す
- 検索条件はあとで追加する

補足:
- Week 1は「骨組み」があれば十分

---

# Week 1で作る主なファイル

- `Application.java`
- `application.yml`
- `docker-compose.yml`
- `schema.sql`
- `entity/Customer.java`
- `repository/CustomerRepository.java`
- `dto/customer/CreateCustomerRequest.java`
- `dto/customer/CustomerResponse.java`
- `service/CustomerService.java`
- `controller/CustomerController.java`
- `exception/ResourceNotFoundException.java`
- `exception/GlobalExceptionHandler.java`

---

# 実装順のおすすめ

1. Spring Bootプロジェクト作成
2. DockerでPostgreSQL起動
3. `application.yml` でDB接続
4. `customers` DDL作成
5. `Customer` Entity作成
6. `CustomerRepository` 作成
7. 登録DTO作成
8. 登録Service作成
9. 登録Controller作成
10. 詳細取得API作成

---

# 動作確認チェックリスト

- `mvn spring-boot:run` で起動する
- `docker compose up -d` でDBが起動する
- DB接続エラーが出ない
- `customers` テーブルが存在する
- `POST /api/customers` が成功する
- DBに登録データが入る
- `GET /api/customers/{customerId}` が動く
- 不正な入力でバリデーションエラーになる

---

# この週で後回しにしてよいもの

- `policies` テーブル
- 契約登録API
- ステータス更新
- CSV取込
- Vue.js
- 認証
- 支払情報テーブル
- 詳細なログ整備
- テストの作り込み

---

# まとめ

- Week 1は `customers` 周りに集中する
- まずは `最小起動 → Hello API → DB接続 → customers実装` の順で進める
- 起動、接続、登録、取得までを確実に終わらせる
- 検索は余力があれば骨組みだけでよい
- 毎日、起動確認とAPI確認を入れる

---

# 次のアクション

1. Spring Bootの初期プロジェクトを作る
2. PostgreSQL用の `docker-compose.yml` を用意する
3. `customers` のDDLを作る
4. `POST /api/customers` 実装に着手する

---

# 確認事項

| 項目 | 現状 |
| --- | --- |
| DB | PostgreSQL前提で記載 |
| 担当 | 自分で進める前提 |
| Week 1の範囲 | Customer機能中心 |
| 検索API | 骨組みまでで可 |
| 本格テスト | Week 3で強化予定 |
