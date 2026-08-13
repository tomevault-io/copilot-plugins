## three-webgpu-webxr-sandbox

> Three.js WebGPURendererでオフスクリーン描画し、XRProjectionLayerにコピーする方式でWebXR描画を行うテストアプリ。Three.js内蔵のXRManager/renderループは使用しない。

# CLAUDE.md - Three.js WebGPU + WebXR Sandbox

## プロジェクト概要
Three.js WebGPURendererでオフスクリーン描画し、XRProjectionLayerにコピーする方式でWebXR描画を行うテストアプリ。Three.js内蔵のXRManager/renderループは使用しない。

## 技術スタック
- TypeScript + Vite
- Three.js (three/webgpu) - WebGPURenderer使用
- WebXR API直接利用 + XRGPUBinding

## アーキテクチャ

### モジュール構成
```
src/
├── core/
│   ├── WebGPUContext.ts    # GPUAdapter/Device管理（現在未使用、Three.jsがDevice作成）
│   ├── XRSessionManager.ts # XRSession, XRGPUBinding, ProjectionLayer管理
│   └── RenderLoop.ts       # 通常/XRデュアルモードループ
├── renderer/
│   ├── ThreeRenderer.ts    # WebGPURenderer wrapper, RenderTarget管理
│   └── XRBlitter.ts        # RenderTarget→XRProjectionLayerコピー
├── scene/
│   └── DemoScene.ts        # テスト用シーン（Box, Sphere, Grid）
├── types/
│   └── webxr-webgpu.d.ts   # XRGPUBinding等の型定義
└── main.ts                 # エントリーポイント
```

### データフロー
```
非XRモード:
  requestAnimationFrame → ThreeRenderer.renderToCanvas()

XRモード:
  xrSession.requestAnimationFrame → XRFrame
    ↓
  For each XRView:
    1. updateCameraFromXRView() - 行列更新
    2. renderToTarget() - RenderTargetに描画
    3. XRBlitter.blit() - copyTextureToTexture
```

## 重要な実装ポイント

### GPUDevice共有
Three.js WebGPURendererが内部で作成するGPUDeviceを取得し、XRGPUBindingに渡す。
```typescript
const device = threeRenderer.getGPUDevice();
xrManager.setDevice(device);
```

### 内部GPUTexture取得（内部API依存）
```typescript
// ThreeRenderer.ts
getRenderTargetGPUTexture(): GPUTexture | null {
  const backend = (this.renderer as any).backend;
  const textureData = backend.get(this.renderTarget.texture);
  return textureData?.texture || null;
}
```

### XRカメラ行列更新
```typescript
camera.projectionMatrix.fromArray(view.projectionMatrix);
camera.matrixWorldInverse.fromArray(view.transform.inverse.matrix);
camera.matrixWorld.copy(camera.matrixWorldInverse).invert();
```

## 開発

```bash
# 依存関係インストール
npm install

# 開発サーバー起動（HTTPS必須）
npm run dev

# ビルド
npm run build
```

## 注意事項
- WebXRはSecure Context必須（HTTPS）
- XRGPUBinding APIは実験的機能
- Three.js内部APIに依存している箇所あり（バージョン互換性注意）
- テクスチャフォーマットはXRProjectionLayerと一致させる必要あり

---
> Source: [WebXR-JP/three-webgpu-webxr-sandbox](https://github.com/WebXR-JP/three-webgpu-webxr-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
