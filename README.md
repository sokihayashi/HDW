# HDW — Hayashi Design Works TOPグラフィック

soki-hayashi.com のTOP（ヒーロー）に iframe で埋め込む p5.js 製のインタラクティブ生成アート。

## ファイル

| ファイル | 内容 |
|---|---|
| `index.html` | **現行の新作「HYDRA」**。SDFレイマーチングの液体生命体。触ると飛び散り、粘って戻る。スマホ縦・タッチ前提。モノトーン＋差し色。 |
| `original.html` | サイトに元々載っていた「雫の積層」グラフィック。リファレンスとして保存。 |

## コンセプト（HYDRA）

WebGLフラグメントシェーダによる SDF レイマーチングで、水銀のような液体生命体を描く。GENUARY 2026 "Resonant Hydra-morphic" をサイトのヒーロー用に再設計したもの。

- **タッチ＝飛散**: タップすると16個の質点が3D方向へ爆ぜ、`smin`（スムーズ最小）で融合しながら粘性をもってゆっくり中心へ戻る
- **横ドラッグ＝回転 / 縦スワイプ＝ページスクロール**（スクロールを殺さない）
- **モノトーン＋差し色**: 基本は白〜グレー。飛び散って「興奮」しているときだけリムに差し色（既定は青）が灯り、収まると白へ
- 端末センサー・マイクは不使用 → Wixのsandbox iframeでも確実に動く（音響反応モードはWixでマイクが使えないため削除）
- 無操作でもゆっくり自転・呼吸し、たまに静かに脈打つ

### 軽量化（重さ対策）
- レイマーチングはGPU負荷が高いので、**低解像度バッファでレンダ → CSSで全画面に拡大**（液体は滑らかなので粗さが目立たない）
- `pixelDensity(1)` 固定、スマホはレンダ解像度をさらに小さく（`CONFIG.renderMaxMobile`）

調整は `index.html` 冒頭の `CONFIG` でOK（差し色・レンダ解像度）。さらに質感はフラグメントシェーダ `fs` 内（背景の明るさ `bgColor`、影/ハイライト、グレイン量など）。

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
