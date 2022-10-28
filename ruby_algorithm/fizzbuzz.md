# 学んだこと

- 最初、returnではなくputsと書いていた。READMEで 「〜を返すプログラム」とあるのをちゃんとみていなかった。
- 1つ目に15の倍数の条件ではなく3や5の倍数の条件を記載してしまうと、15の倍数が引数に渡された場合も3や5の条件が通ってしまい思うような結果が得られない。

- 最初に下記のように書いていた
```
def fizz_buzz(number)
   if number % 15 == 0
    return 'Fizz Buzz'
  elsif number % 3 == 0
    return 'Fizz'
  elsif number % 5 == 0
    return 'Buzz'
  else
    return number.to_s
  end
end

```
mintestは通ったが、Lintチェック後に修正をし、以下のように変更

```
def fizz_buzz(number)
  return 'Fizz Buzz' if number % 15 == 0
  return 'Fizz' if number % 3 == 0
  return 'Buzz' if number % 5 == 0

  number.to_s
end
```
以後「return ~ if」という書き方をする点や、elsifを繰り返すとコードの行数が増えて見辛くなる点を気をつける。
