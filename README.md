# kikuvi-skills-ja

[Kikuvi](https://kikuvi.com) の APM パッケージです。KikuviのSkillsとMCPサーバーをまとめて導入できます。

## インストール

### Claude Code
```text
/plugin marketplace add Kikuvi-Inc/kikuvi-skills-ja
/plugin install kikuvi-skills-ja@kikuvi
```

### Claude app
Customize → Personal Plugins → Create plugins -> Add marketplace -> Add from a repository をクリックし、`Kikuvi-Inc/kikuvi-skills-ja` を追加し、
`kikuvi-skills-ja` を sync します。

初回接続時にブラウザが開き、Kikuvi アカウントへの OAuth サインインが完了します。

## 含まれるもの

| 種類 | 名前 | 説明 |
|------|------|------|
| MCP サーバー | `kikuvi` | Kikuvi API へのアクセス（ヒアリング設計・セッション・レポートなど） |
| スキル | `kikuvi-skills-ja` | ヒアリング設計・管理・分析のワークフローガイド |

## 必要要件
- [Kikuvi](https://kikuvi.com) アカウント
