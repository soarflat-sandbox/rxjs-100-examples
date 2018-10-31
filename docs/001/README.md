# 001

[「RxJS でひとまとめの複数の非同期処理のうち一つでもエラーになったら他の全ての非同期処理を破棄するコードを書いた」](https://qiita.com/ovrmrw/items/1fbbb92f2c766734c752)を参考にして書いたサンプルコード

`Observable.forkJoin` がどのような挙動をするのか理解するために書いた。

## サンプル

### サンプル 1

[Demo](index.html)

利用しているオペレーター

- Observable.from
- Observable.catch
- Observable.empty
- Observable.range
- Observable.forkJoin
- Observable.defaultIfEmpty
- Observable.zip

```js
const Observable = Rx.Observable;

function request(value) {
  // Promise（9割の確率で resolve し、1割の確率で reject する）を Observable に変換する。
  // 変換した Promise が resolve した場合、resolve した値を Observable.of() で変換した Observable が返される（value.toUpperCase() の戻り値が流れる）。
  return (
    Observable.from(
      new Promise((resolve, reject) => {
        if (Math.random() > 0.1) {
          setTimeout(() => {
            resolve(value.toUpperCase());
          }, Math.floor(Math.random() * 1000));
        } else {
          reject();
        }
      })
    )
      // 変換した Promise が reject した場合、error イベントが発火されるため、
      // Observable.catch で error を拾い Observable.empty を返す。
      .catch(() => Observable.empty())
  );
}

Observable.range(0, 10)
  .mergeMap(i => {
    return (
      Observable
        // 配列内の Promise から生成した Observable が全て値を流した（resolve して complete イベントが発火した）場合、次のオペレータに値を流す。
        // 今回の場合流れる値は ['h', 'e', 'l', 'l', 'o']
        // 1つでも error イベントを発火した（reject した）場合、resolve した Observable も破棄して error イベントを発火する。
        .forkJoin([request('h'), request('e'), request('l'), request('l'), request('o')])
        // ['h', 'e', 'l', 'l', 'o'] => 'hello'
        .map(array => array.join(''))
        // Observable.forkJoin を実行時に error イベントが発火した場合実行される。'empty'を流す。
        .defaultIfEmpty('empty')
        // 流れてきた値（'hello'か'empty'）と Observable.of(i) を一つの配列にして値を流す
        // 今回の場合流れる値は ['HELLO', Observable.of(i)の結果] か ['empty', Observable.of(i)の結果]
        .zip(Observable.of(i))
        .map(([value, index]) => `${index}: ${value}`)
    );
  })
  .subscribe(console.log);
```
