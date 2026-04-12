# PicoClaw Helm Chart

PicoClaw Helm Chartは、軽量なAIアシスタントである[PicoClaw](https://github.com/sipeed/picoclaw)をKubernetesクラスターにデプロイするためのHelmチャートです。

- チャートバージョン: 0.1.11
- PicoClaw バージョン (appVersion): v0.2.5

[English version](README.md)

## 前提条件

- Kubernetesクラスター
- Helm 3.0以上

## インストール

以下のコマンドを実行してHelmリポジトリを追加し、チャートをインストールします。

```bash
helm repo add picoclaw https://turtton.github.io/picoclaw-helm
helm repo update
helm install my-picoclaw picoclaw/picoclaw --version 0.1.11
```

## 設定

`values.yaml` で設定可能な主要なパラメータは以下の通りです。

| パラメータ | 説明 | デフォルト値 |
|-----------|-------------|---------|
| `image.repository` | PicoClawのイメージリポジトリ | `docker.io/sipeed/picoclaw` |
| `image.tag` | PicoClawのイメージタグ（デフォルトは`appVersion`） | `""` |
| `config.agents` | エージェントの設定（デフォルト値、リスト） | `values.yaml`を参照 |
| `config.model_list` | モデルプロバイダーの設定 | `[]` |
| `config.channels` | 通信チャネルの設定（Discordなど） | `values.yaml`を参照 |
| `securityConfig` | `.security.yml`の内容（APIキーやトークンなど）をインラインで定義 | `{}` |
| `securitySecret.existingSecret` | `.security.yml`を含む既存のSecret名 | `""` |
| `workspaceFiles` | `PICOCLAW_HOME/workspace/`に配置するファイル | `{}` |
| `skillFiles` | ConfigMapからコピーするスキルファイルの定義 | `{}` |
| `persistence.enabled` | `PICOCLAW_HOME`の永続ボリュームを有効化 | `true` |
| `persistence.size` | 永続ボリュームのサイズ | `5Gi` |
| `launcher.enabled` | オプションのLauncher WebUIサイドカーを有効化 | `false` |
| `launcher.port` | Launcher WebUIのポート | `18800` |
| `copilotCli.enabled` | GitHub Copilot CLIサイドカーを有効化 | `false` |
| `service.type` | Kubernetes Serviceのタイプ | `ClusterIP` |
| `service.port` | Kubernetes Serviceのポート | `18790` |
| `resources` | Podのリソースリクエストと制限 | `{}` |
| `nodeSelector` | ノード選択ラベル | `{}` |
| `tolerations` | PodのTolerations | `[]` |
| `affinity` | PodのAffinityルール | `{}` |

## セキュリティ設定

APIキーやボットトークンなどの機密情報が必要な場合、`.security.yml`ファイルを通じてPicoClawに提供できます。以下の2つの方法があります。

1. **`securityConfig`**: `values.yaml`に直接セキュリティ設定を記述します。チャートが自動的にSecretを作成します。
2. **`securitySecret.existingSecret`**: `.security.yml`の内容を含む既存のSecretを使用します。デフォルトでは、Secret内の`security.yml`というキーにYAMLデータが含まれている必要があります。

## ワークスペースとスキル

起動時にカスタムのワークスペースファイルやスキルファイルをPicoClawに配置できます。

- **`workspaceFiles`**: キーをファイル名、値を内容とするマップです。これらのファイルは `/root/.picoclaw/workspace/` にコピーされます。
- **`skillFiles`**: 指定したConfigMapからスキル定義を `/root/.picoclaw/workspace/skills/<name>/` にコピーします。

`skillFiles` の設定例:

```yaml
skillFiles:
  my-skill:
    configMapName: my-skill-configmap
    items:
      - key: SKILL.md
        path: SKILL.md
```

## Launcher WebUI

Launcherサイドカーを有効にすることで、WebUIを利用できます。

```yaml
launcher:
  enabled: true
```

Launcherは、PicoClawゲートウェイへのリクエストをプロキシするWebインターフェースを提供します。デフォルトでは、専用のイメージタグ（`launcher`）を使用します。

## アンインストール

リリース `my-picoclaw` をアンインストールするには、以下のコマンドを実行します。

```bash
helm uninstall my-picoclaw
```

## 上流プロジェクト

PicoClawの詳細については、[公式リポジトリ](https://github.com/sipeed/picoclaw)を参照してください。
