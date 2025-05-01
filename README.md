<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ひかちゃんクーポン</title> <! --ここが正しいtitleタグ -->
  <style>
    body {
      font-family: "Hiragino Maru Gothic ProN", Meiryo, sans-serif;
      background: #fff0f5;
      text-align: center;
      padding: 40px;
      color: #555;
    }

    .coupon-box {
      background: #fff;
      border: 3px dotted #f4a7b9;
      border-radius: 20px;
      padding: 30px;
      display: inline-block;
      box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
      position: relative;
    }

    .ribbon {
      width: 120px;
      height: 30px;
      background: #f4a7b9;
      color: white;
      font-weight: bold;
      line-height: 30px;
      position: absolute;
      top: -15px;
      left: calc(50% - 60px);
      border-radius: 15px;
      box-shadow: 0 3px 5px rgba(0, 0, 0, 0.2);
    }

    h1 {
      color: #e75480;
      font-size: 26px;
      margin-top: 30px;
    }

    p {
      font-size: 18px;
      line-height: 1.6;
    }
  </style>
</head>
<body>
  <div class="coupon-box">
    <div class="ribbon">ひみつクーポン</div>
    <h1>ひかちゃん用</h1>
    <p>この画面を見たらはるちゃんにQRコードを表示してもらってください
      <br>最悪の事態が起こります</p>
  </div>
</body>
</html>
