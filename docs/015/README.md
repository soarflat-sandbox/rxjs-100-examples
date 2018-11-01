# 015

チェックボックスの選択数を View に反映するサンプルコード。

## サンプル

### サンプル 1

[Demo](index.html)

利用しているオペレーター

- Observable.fromEvent
- Observable.map
- Observable.startWith
- Observable.scan

```html
<!DOCTYPE html>
<html lang="ja">

<head>
  <meta charset="UTF-8">
  <title>015</title>
</head>

<body>
  <p>選択: <span id="countSelected">0</span>個</p>
  <input type="checkbox" checked />
  <input type="checkbox" />
  <input type="checkbox" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/rxjs/5.5.10/Rx.min.js"></script>
  <script>
    const $count = document.getElementById('countSelected');
    const $checkboxes = document.querySelectorAll('input[type="checkbox"]');
    const countSelected = document.querySelectorAll('input[type="checkbox"]:checked').length;

    Rx.Observable
      .fromEvent($checkboxes, 'input')
      .map(e => e.target.checked ? 1 : -1)
      .startWith(countSelected)
      .scan((acc, value) => acc + value, 0)
      .subscribe(value => $count.innerText = value);
  </script>
</body>

</html>
```
