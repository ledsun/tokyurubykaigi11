# [fit] Rspecと非同期関数テスト

株式会社ラグザイア 中島 滋
@TokyuRubyKaigi 2017/07/29

---

## 自己紹介

- 中島 滋 a.k.a. ledsun
- 受託開発でWebアプリケーションを開発
- Rails経験半年

---

## 最初に大事なこと

- RSpecで非同期関数をテストする
良い方法を教えて欲しい
- RSpecでなくてもRubyで
非同期関数をテストする...


---

## わかっていること

- RSpecはテスト失敗の収集に例外を使う
- Rubyではサブスレッド内の例外は
がんばらないと捕まえられない

---

## 悩んでいること
非同期関数
特にスレッドを使って実装
どうテストする？

---

## テストしたい関数の実装イメージ

```rb
def async_func
  Thread.new do
    # 何か時間がかかる処理
    yield result
  end
end
```

```rb
async_func do |result|
  # 結果をもらって何かする
end
```

---

## やりたいテスト

```rb
it '非同期関数の失敗を捕まえられること' do
  async_func do |result|
    expect(result).to eq(expected_value) #expectの失敗 => 例外
  end
end
```

async_funcの中は別スレッド
別スレッドの例外をキャッチできない

---
## 困ること

expect失敗を検知できない

---

## メインスレッドに結果を返すには？

- スレッド変数を返して
呼び出し元でjoin
- 例外キャッチして
メインスレッドに向かってraise

---

## プロダクトコードに特定の実装を強制する？
- プロダクトコードの実装ミスったら？
- 検知できない
- テストコードの意味？


---

## できそうなこと

- RSpecの例外収集機構を書き換える
failure_notifierなど
- 非同期関数をスレッド以外で書き直す
Fiber？
- 非同期関数の結果をメインスレッドに戻す
メインスレッドでexpect

---

## できそう？

RSpecに手を入れるのは大変そう
Fiberよくわからない、こわい

---

## 非同期関数の結果をメインスレッドに戻す

```rb
it '非同期関数の失敗を捕まえられること' do
  result = async_play do |curtail|
    async_func do |result|
      curtail.call result
    end
  end
  expect(result).to eq(expected_value)
end
```

これならテストできそう

---

## キューを使う

```rb
def async_play
  q = Queue.new
  yield (reuslt) -> { q.push result }
  q.pop
end
```

こんな感じで実装

---

## 試したかったらgemある

`gem 'async_play'`


```rb
results = AsyncPlay.opening{ | curtain | Thread.new { curtain.call 1 } }
```

## 詳細は

http://qiita.com/ledsun/items/0e1dd4ece43dc56653c7

---

## もっと良い方法があれば教えてください
