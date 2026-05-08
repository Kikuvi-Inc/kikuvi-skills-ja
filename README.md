# kikuvi-skills-ja

[Kikuvi](https://kikuvi.com) の APM パッケージです。1コマンドで MCP サーバーと Skills をセットアップできます。

## インストール

```bash
apm install Kikuvi-Inc/kikuvi-skills-ja
```

初回接続時にブラウザが開き、Kikuvi アカウントへの OAuth サインインが完了します。

## 含まれるもの

| 種類 | 名前 | 説明 |
|------|------|------|
| MCP サーバー | `kikuvi` | Kikuvi API へのアクセス（ヒアリング設計・セッション・レポートなど） |
| スキル | `kikuvi-skills-ja` | ヒアリング設計・管理・分析のワークフローガイド |

## 必要要件

- [Kikuvi](https://kikuvi.com) アカウント
- APM がインストールされていること

### APM のインストール

**macOS / Linux**
```bash
curl -sSL https://aka.ms/apm-unix | sh
```

**Windows**
```powershell
irm https://aka.ms/apm-windows | iex
```

詳細は [APM ドキュメント](https://microsoft.github.io/apm/) を参照してください。
