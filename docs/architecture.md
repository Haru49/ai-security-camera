# アーキテクチャ方針

## 結論

MVPはKotlinネイティブAndroidアプリと、MacBook Air上のPython AIサーバーで作る。

- UI、カメラ、通知、ローカル保存、フレーム送信: Kotlin
- カメラ映像取得: CameraX
- AI解析サーバー: MacBook Air
- サーバー実装: Python + FastAPI
- 骨格推定: Pythonサーバー側のMediaPipe Pose
- 怪しさ判定: 最初はPythonサーバー側のルールベース処理
- 通信: 同一Wi-Fi内のHTTP通信

## PythonをMVPのAndroid内に入れない理由

Android端末上でPythonを動かす構成は、依存関係、配布、性能、バッテリー、カメラフレーム連携が複雑になりやすい。

このアプリはリアルタイムカメラ処理が中心なので、最初はAndroid公式寄りの構成に寄せる。

Pythonは次の用途で使う。

- MacBook Air上のAI解析サーバー
- 特徴量設計の実験
- 類似度計算の検証
- 将来のサーバーサイド処理
- モデル改善やユーザーフィードバック学習の検証

## AI実行場所

MVPではAIをAndroid端末内ではなく、同一Wi-Fi内のMacBook Airで動かす。

理由:

- 古いAndroidスマホではリアルタイム推論、発熱、バッテリー、FPSが厳しくなりやすい
- Python、MediaPipe、OpenCVをMacBook Air側で扱う方が実験しやすい
- まずは「Androidで撮る」「サーバーで解析する」「Androidに結果を返す」という全体の流れを早く作れる
- MacBook Airで性能が足りなければ、Windows PC、GPU付きPC、Jetson、クラウドへ移行しやすい

初期構成:

```text
Androidスマホ
  CameraXで撮影
  1秒に1枚程度のJPEGを送信
        ↓ HTTP
MacBook Air
  Python FastAPI
  MediaPipe Pose
  特徴量抽出
  怪しさスコア計算
        ↓ JSON
Androidスマホ
  結果表示
  通知
  ローカル保存
```

Windows PCは、MacBook Airで処理性能が足りない場合の移行先候補にする。

## MVP処理フロー

1. Androidアプリを起動する
2. CameraXでカメラプレビューを表示する
3. ImageAnalysisでフレームを取得する
4. 1秒に1枚程度のJPEGをMacBook AirのPythonサーバーへ送る
5. PythonサーバーでMediaPipe Poseを実行する
6. 人物の骨格ランドマークを取得する
7. ランドマークから簡易特徴量を計算する
8. 滞在時間、停止時間、移動量から怪しさスコアを計算する
9. JSONでAndroidへ解析結果を返す
10. スコアが閾値以上ならAndroid側で通知・保存する

## 保存データ

MVPではRoomまたはJSON Linesでローカル保存する。

最初は実装速度を優先してJSON Linesでもよいが、Androidアプリとして継続開発するならRoomを第一候補にする。

保存対象:

- 検知イベント
- タイムスタンプ
- 怪しさスコア
- 骨格ランドマーク
- 簡易特徴量
- 静止画または短い動画へのローカルパス

## 特徴量の初期案

最初は個人特定に寄りすぎない、姿勢・動き中心の特徴量にする。

- 画面内の人物中心座標
- 肩幅に対する体の高さ比
- 肩、腰、膝、足首の相対位置
- フレーム間の移動量
- 停止時間
- 滞在時間
- 方向転換回数
- 画面内の指定エリアへの接近

## 再訪問検知の扱い

再訪問検知はMVP後に実装する。

最初は「同一人物の断定」ではなく、「過去に怪しい行動をした人物特徴と似ている」というスコアとして扱う。

比較方法の初期案:

- 特徴ベクトルのコサイン類似度
- 正規化ユークリッド距離
- 複数フレーム特徴量の平均・分散比較
- 服装色などの補助特徴は後段で追加

## 初期ディレクトリ構成案

```text
app/
  src/main/java/...
    camera/
    network/
    scoring/
    storage/
    notification/
    ui/
server/
  app/
    main.py
    pose/
    scoring/
    schemas/
docs/
  architecture.md
  git-notes.md
```

## 参考

- MediaPipe Pose Landmarker Android: https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker/android
- CameraX: https://developer.android.com/media/camera/camerax
