# HDW — Hayashi Design Works TOPグラフィック

soki-hayashi.com のTOP（ヒーロー）に iframe で埋め込む p5.js 製のインタラクティブ生成アート。

## ファイル

| ファイル | 内容 |
|---|---|
| `index.html` | **現行の新作「HYDRA」**。SDFレイマーチングの液体生命体。触ると飛び散り、粘って戻る。スマホ縦・タッチ前提。モノトーン＋差し色。 |
| `original.html` | サイトに元々載っていた「雫の積層」グラフィック。リファレンスとして保存。 |

## コンセプト（HYDRA）

WebGLフラグメントシェーダによる SDF レイマーチングで、水銀のような液体生命体を描く。GENUARY 2026 "Resonant Hydra-morphic" をサイトのヒーロー用に再設計したもの。

- **タップ＝飛散**: 16個の質点が3D方向へ爆ぜ、`smin`（スムーズ最小）で融合しながら粘性をもってゆっくり中心へ戻る
- **ドラッグ＝掴んで引く**: body が指へ寄り（バネ＋減衰で遅れて追従＝液体の慣性）、引いた向きへ尾を引き、速くなぞると表面が波立つ。横移動が主のときは回転も加わる
- **縦スワイプ＝ページスクロール**（スクロールを殺さない。`touch-action:pan-y` ＋ `preventDefault` しない）
- **モノトーン＋差し色**: 白〜グレーの液体クローム。縁（フレネル）に差し色が常時うっすら、飛散して「興奮」しているほど強く灯る
- マテリアルはスペキュラ＋擬似環境反射＋フレネルで水銀のような質感。背景はスタジオ風グラデ＋ビネット
- 端末センサー・マイクは不使用 → Wixのsandbox iframeでも確実に動く（音響反応モードはWixでマイクが使えないため削除）
- 無操作でもゆったり自転・呼吸し、たまに大きく伸び上がる

### 解像度と軽さ（適応解像度）
- レイマーチングはGPU負荷が高いので、**低解像度バッファでレンダ → CSSで全画面に拡大**
- **適応解像度**: 実FPSを監視し、軽ければ解像度を上げ（最大で表示画素相当までクッキリ）、重ければ下げて60fpsを維持。`CONFIG.res` で範囲を調整
- `pixelDensity(1)` 固定

調整は `index.html` 冒頭の `CONFIG`（差し色・解像度範囲）。質感はフラグメントシェーダ `fs` 内（背景グラデ、`env()`、スペキュラ/フレネル、グレイン量など）。

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
