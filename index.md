% resque-scheduler
% yamotonalds
% 2016-02-24

# resque-scheduler

## resque-schedulerとは

* 指定日時にジョブを実行するresqueの拡張ライブラリ
* 2種類
    * Delayed … 指定日時に1度だけ実行
    * Recurring … 定期実行

# resqueの使い方

## Job登録

```
Resque.enqueue(AwesomeJob, args）
```

# Delayedの使い方

## Job登録

```
Resque.enqueue_at(5.days.from_now, AwesomeJob, args)
```

## DelayedJob削除

```
# Delayedの削除
Resque.remove_delayed(AwesomeJob, args)
```

. . .

argsは完全一致

## DelayedJob条件指定削除

```
# Delayedの削除
Resque.remove_delayed_selection { |args| args[0][:id] == 1 }
```

## なんかハマった(Delayed)

- class or string

# Recurringの使い方

## Job登録

```
# 静的
Resque.schedule = YAML.load_file('your_resque_schedule.yml')

# 動的（要設定）
config = {}
config[:class] = 'AwesomeJob'
config[:args] = 'foo bar'
config[:every] = ['1h', {first_in: 5.minutes}]
#config[:cron] = '0 0 * * *'
config[:persist] = true
Resque.set_schedule(schedule_id, config)
```

[YAML例](https://github.com/resque/resque-scheduler#static-schedules)

## なんかハマった(Recurring)

- ✕ `Resque::Scheduler.dynamic = true`
- ◯ `DYNAMIC_SCHEDULE=1`

## RecurringJob削除

```
# Recurringの削除
Resque.remove_schedule(schedule_id)
```

# 仕組み

## プロセスとポーリング

resque-scheduler用にプロセスが1つ必要。

(rake resque:scheduler)

そのプロセスがポーリングして時間になっていたらジョブをワーカーに渡す。

## データ構造(Delayed)

- delayed_queue_schedule…SortedSet(timestamp, timestamp)
    - ソート済み実行日時
    - delayed_queue_peekメソッドで取得
- timestamps:#{encoded_job}…Set(delayed:#{timestamp})
    - ジョブがいつ実行することになっているか
- delayed:#{timestamp}…List(#{encoded_job})
    - timestampに実行するジョブ情報のリスト
    - delayed_timestamp_peekメソッドで取得

. . .

READMEに説明が無い

## その他

- [管理画面](https://github.com/resque/resque-scheduler#resque-web-additions)
- Hooks
- resque-status対応

# ActiveJob

## 使い方

```
# Resque.enqueue
MyJob.perform_later(record)

# Resque.enqueue_at (Delayed)
MyJob.set(wait_until: 5.days.from_now).perform_later(record)

# Resque.set_schedule (Recurring)
# 無し

# 実行待ちジョブ情報取得
# 無し
```

## 良い所

- デフォルトで使える
- Mailerと統合済み
- オブジェクトをそのまま受け渡しできる
- [バックエンドが選べる](https://github.com/rails/rails/blob/v4.2.5.1/activejob/lib/active_job/queue_adapters.rb)
    - Resque
    - Sidekiq

## 悪い所

- Recurring（定期実行）が無い
- 実行待ちジョブを取得する[インターフェースが無い](https://github.com/rails/rails/blob/v4.2.5.1/activejob/lib/active_job/queue_adapters/resque_adapter.rb)
- 実行待ちをキャンセルするインターフェースが無い

. . .

Gemまで調べてない

## PRチャンスでは？

![](images/pro.png)

とあるプロ

# まとめ

##

- ActiveJobで済むならActiveJobで
    - すっきり書ける
- 定期実行や実行待ちジョブの管理が必要ならresque-schedulerは悪く無さそう
    - ちょっと癖がある
    - 軽くWrapして使うのが良さそう
    - 既存ジョブをそのまま使えるのが良い
