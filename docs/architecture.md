# アーキテクチャ方針

## 結論

MVPはKotlinネイティブAndroidアプリとして作る。

- UI、カメラ、通知、ローカル保存: Kotlin
- カメラ映像取得: CameraX
- 骨格推定: MediaPipe Pose Landmarker for Android
- 怪しさ判定: Kotlin内のルールベース処理
- Python: MVPでは端末内に入れず、後段の分析・実験・サーバー用途に分離

## PythonをMVPのAndroid内に入れない理由

Android端末上でPythonを動かす構成は、依存関係、配布、性能、バッテリー、カメラフレーム連携が複雑になりやすい。

このアプリはリアルタイムカメラ処理が中心なので、最初はAndroid公式寄りの構成に寄せる。

Pythonは次の用途で使う。

- 保存済み骨格データの分析
- 特徴量設計の実験
- 類似度計算の検証
- 将来のサーバーサイド処理
- モデル改善やユーザーフィードバック学習の検証

## MVP処理フロー

1. Androidアプリを起動する
2. CameraXでカメラプレビューを表示する
3. ImageAnalysisでフレームを取得する
4. MediaPipe Pose Landmarkerにフレームを渡す
5. 人物の骨格ランドマークを取得する
6. ランドマークから簡易特徴量を計算する
7. 滞在時間、停止時間、移動量から怪しさスコアを計算する
8. スコアが閾値以上ならイベントとして保存する
9. Android通知を出す

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
    pose/
    scoring/
    storage/
    notification/
    ui/
docs/
  architecture.md
  git-notes.md
```

## 参考

- MediaPipe Pose Landmarker Android: https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker/android
- CameraX: https://developer.android.com/media/camera/camerax
