# HDW — Hayashi Design Works TOPグラフィック

soki-hayashi.com のTOP（ヒーロー）に iframe で埋め込む p5.js 製のインタラクティブ生成アート。

## ファイル

| ファイル | 内容 |
|---|---|
| `index.html` | **現行の新作「INK」**。指でなぞると水中にインクが広がる、スマホ縦・タッチ前提のインタラクティブ作品。モノトーン＋差し色。 |
| `original.html` | サイトに元々載っていた「雫の積層」グラフィック。リファレンスとして保存。 |

## コンセプト（INK）

指でなぞった軌跡と速度に沿って、インクが水中に滲み広がる。

- **タッチ主役（マルチタッチ）**。端末の傾きセンサーは使わない → 後述のWix制約下でも確実に動く
- **モノトーン＋差し色**: 速い／新しいインクの芯だけ差し色（既定は青）が灯り、落ち着くと白へ
- **軽量実装**: 重い流体シミュではなく「ソフトなスプライト＋ノイズ流れ場＋加算合成＋ゆっくり拡散」で滲みを再現
- 無操作でも環境インクが漂い、ヒーローとして生き続ける
- **ダブルタップでクリア**

調整は `index.html` 冒頭の `CONFIG` だけ触ればOK（差し色・消える速さ・粒数・水流の強さ など）。

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
