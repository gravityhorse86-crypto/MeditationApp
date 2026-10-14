# DB定義

Supabase (PostgreSQL)。認証は Supabase Auth（`auth.users`）を使う。

## 方針
- 全テーブルで RLS（行レベルセキュリティ）を有効にし、`user_id = auth.uid()` の行だけ読み書きできるようにする
- `user_id` は `auth.users(id)` への外部キー。**`ON DELETE CASCADE`**（ユーザーを消すと、その人の全データが消える）
- 入力値のチェックは DB ではなく、コード側の Zod で行う（AI出力のチェックも同じ）
- `updated_at` はトリガーを使わず、更新時にアプリ側で現在時刻を入れる
- `user_settings` の行は事前に作らない。行がなければ BGM は ON とみなし、設定を保存するときに upsert する

## user_settings（ユーザー設定）

| カラム | 項目名 | 型 | NOT NULL | PK/FK | デフォルト | 備考 |
|---|---|---|---|---|---|---|
| user_id | ユーザーID | uuid | ○ | PK, FK | | auth.users.id を参照 |
| bgm_enabled | BGM ON/OFF | boolean | ○ | | true | true: ON / false: OFF |
| created_at | 作成日時 | timestamptz | ○ | | now() | |
| updated_at | 更新日時 | timestamptz | ○ | | now() | 更新時にアプリ側で書き換える |

## meditations（瞑想プログラム）

| カラム | 項目名 | 型 | NOT NULL | PK/FK | デフォルト | 備考 |
|---|---|---|---|---|---|---|
| id | 瞑想ID | uuid | ○ | PK | gen_random_uuid() | |
| user_id | ユーザーID | uuid | ○ | FK | | auth.users.id を参照 |
| mood | 現在の気分 | varchar(30) | ○ | | | 下記の6種類のどれか |
| duration_minutes | 設定時間 | integer | ○ | | | 3〜30（分） |
| guide_level | ガイド量 | varchar(20) | ○ | | | 少なめ / 普通 / 多め |
| meditation_type | 瞑想タイプ | varchar(50) | ○ | | | 呼吸法、ボディスキャン等（AIが決める） |
| title | 瞑想タイトル | varchar(100) | ○ | | | AI生成でも可 |
| guide_content | ガイド内容 | jsonb | ○ | | | 下記の形式 |
| created_at | 作成日時 | timestamptz | ○ | | now() | |

### mood の選択肢（この文字列をそのまま保存する）
1. 不安だ
2. 疲れている
3. イライラする
4. ソワソワして落ち着かない
5. 気分が沈む
6. 該当するものはなし

### guide_content の形式
```json
{
  "segments": [
    { "start_sec": 0, "text": "ゆっくり息を吸って" },
    { "start_sec": 12, "text": "吐く息と一緒に力を抜いて" }
  ]
}
```
- `start_sec`: 再生開始から何秒目に読むか
- `start_sec` は小さい順に並べる
- 最後の `start_sec` は、設定時間（秒）を超えない
- 上の2つは保存前に Zod で検査する

## meditation_sessions（瞑想履歴）

| カラム | 項目名 | 型 | NOT NULL | PK/FK | デフォルト | 備考 |
|---|---|---|---|---|---|---|
| id | 履歴ID | uuid | ○ | PK | gen_random_uuid() | |
| user_id | ユーザーID | uuid | ○ | FK | | auth.users.id を参照 |
| meditation_id | 瞑想ID | uuid | ○ | FK | | meditations.id を参照（CASCADE） |
| started_at | 瞑想開始日時 | timestamptz | ○ | | now() | |
| ended_at | 瞑想終了日時 | timestamptz | | | | 中断時は NULL 可 |
| actual_seconds | 実施時間 | integer | ○ | | 0 | 実際に瞑想した秒数 |
| completed | 完了フラグ | boolean | ○ | | false | 最後まで実施したか |