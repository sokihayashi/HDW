# HDW — Hayashi Design Works TOPグラフィック

soki-hayashi.com のTOP（ヒーロー）に iframe で埋め込む p5.js 製のインタラクティブ生成アート。

## ファイル

| ファイル | 内容 |
|---|---|
| `index.html` | **現行の新作「HYDRA」**。SDFレイマーチングの液体生命体。触ると飛び散り、粘って戻る。スマホ縦・タッチ前提。モノトーン＋差し色。 |
| `original.html` | サイトに元々載っていた「雫の積層」グラフィック。リファレンスとして保存。 |
| `script-shadow.html` | **写真の陰影を台本の文字に置き換える**タイポグラフィ・ポートレート生成ツール。暗い所ほど大きな黒文字、明るい所は小さく/空白に。和文Google Fontsを多数サブセット読み込みして切替。人物切り抜き（背景白）画像を読み込み、PNG書き出し可。 |

## コンセプト（HYDRA）

WebGLフラグメントシェーダによる SDF レイマーチングの **透明ガラス生命体**。背後に置いたブランドのタイポを **屈折・色収差（分光）で歪ませる**＝ロゴとの関わりを持たせる（YouFab / Liquid Modulator 的アプローチ）。GENUARY 2026 "Resonant Hydra-morphic" を土台に再設計。

- **ガラスの屈折**: body 越しに背後の文字（`CONFIG.bgLines`）がレンズのように歪み、縁では色収差で色が割れる。中心は屈折・縁は反射（フレネル）
- **タップ／連打＝どんどん細かく霧化**: タップごとに「細かさ(`fine`)」が上がり、質点が縮み・融合が弱まり・高周波で表面が刻まれて細かい飛沫になる。手を止めるとゆっくり再合体
- **ドラッグ＝掴んで引く**: body が指へ寄り（バネ＋減衰＝液体の慣性）、引いた向きへ尾を引く。横移動が主のときは回転も加わる
- **縦スワイプ＝ページスクロール**（`touch-action:pan-y` ＋ `preventDefault` しない）
- **モノトーン＋差し色**: 黒い艶ガラス（`glassTint`）。縁に差し色が灯り、興奮するほど強く。スタジオ照明の鋭い映り込み＋疑似ブルーム＋S字トーンカーブで締める
- 端末センサー・マイク不使用 → Wixのsandbox iframeでも動く
- 無操作でもゆったり自転・呼吸し、たまに伸び上がる

> 背景の文字が **上下逆さ** に見えたら `CONFIG.flipBg` を `false` に（p5の2Dテクスチャ向きの差異対策）。

### 解像度と軽さ（適応解像度）
- レイマーチングはGPU負荷が高いので、**低解像度バッファでレンダ → CSSで全画面に拡大**
- **適応解像度**: 実FPSを監視し、軽ければ解像度を上げ（最大で表示画素相当までクッキリ）、重ければ下げて60fpsを維持。`CONFIG.res` で範囲を調整
- `pixelDensity(1)` 固定

調整は `index.html` 冒頭の `CONFIG`（差し色・解像度範囲・背景の文字 `bgLines`/`bgFoot`・ガラスの透け `glassTint`・屈折 `lens`・色収差 `dispersion`）。さらに質感はフラグメントシェーダ `fs` 内（`env()` のスタジオ照明、スペキュラ/フレネル、グレイン量など）。

## Wix iframe の制約（リサーチ結果）

Wixの「HTML埋め込み」は別オリジンの **sandbox付きiframe** 内で動くため:

- ❌ **加速度・ジャイロ（DeviceOrientation/DeviceMotion）は使えない**
  - クロスオリジンiframeでは親に `allow="accelerometer; gyroscope"` が必要だがWixでは付与不可
  - iOS Safariの `DeviceOrientationEvent.requestPermission()` も第三者iframe内では実質不可（[WebKit #221399](https://bugs.webkit.org/show_bug.cgi?id=221399)）
- ❌ 親ページDOMへのアクセス不可 / localStorage・Cookieは制限されがち / 音声自動再生・全画面は制限
- ❌ iframeは固定サイズ（中身に合わせた自動リサイズなし）→ 枠内で完結する設計が前提
- ✅ **タッチ/マルチタッチ・Canvas/WebGL・p5.js・requestAnimationFrame は問題なく動く**

→ そのため本作は **タッチ（指）だけ** をインタラクションの主役にしている。

参考: [Wix Studio: HTML iFrameの追加](https://support.wix.com/en/article/studio-editor-adding-an-html-iframe-element) /
[Velo: HTML iframe要素](https://dev.wix.com/docs/develop-websites/articles/wix-editor-elements/other-elements/html-i-frame-element/working-with-the-html-iframe-element)

## プレビュー

ローカルで開くだけで動く（p5.jsはCDN読み込み）。スマホ実機で見たい場合は同一LAN内でサーバを立ててアクセス。

```bash
python3 -m http.server 8000
# PC:    http://localhost:8000/index.html
# スマホ: http://<PCのIP>:8000/index.html
```

本番では `index.html` を Wix の HTML iframe に貼り付けて埋め込む。
