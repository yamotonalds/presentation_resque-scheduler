% resque-scheduler
% yamotonalds
% 2016-02-24

# resque-scheduler

## resque-schedulerとは

* 指定日時にジョブを実行するresqueの拡張ライブラリ
* 2種類
    * Delayed … 指定日時に1度だけ実行
    * Recurring … 定期実行

## resqueの使い方

```
Resque.enqueue(AwesomeJob, args）
```

## resque-schedulerの使い方(delayed)

```
Resque.enqueue_at(5.days.from_now, AwesomeJob, オプション)
```

## resque-schedulerの使い方(scheduled)

```
# 静的
Resque.schedule = YAML.load_file('your_resque_schedule.yml')

# 動的（要設定）
config = {}
config[:class] = 'AwesomeJob'
config[:args] = 'foo bar'
config[:every] = ['1h', {first_in: 5.minutes}]
config[:persist] = true
Resque.set_schedule(schedule_id, config)
```

## resque-schedulerの使い方(削除)


# 仕組み

## プロセスとポーリング

resque-scheduler用にプロセスが1つ必要。

そのプロセスがポーリングして時間になっていたらジョブをワーカーに渡す。

## データ構造

データ構造
（同じ秒だと順番が未定義？）

# ActiveJob

# まとめ
