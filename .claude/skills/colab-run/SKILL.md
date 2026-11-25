---
name: colab-run
description: Google Colab CLI で本プロジェクトの実験スクリプトを GPU 実行し、結果をローカルに回収する。Colab での実行・評価・GPU 利用が必要な時に使用。
---

# Colab 実行手順

## 前提

- `colab` が無ければ `uv tool install google-colab-cli` でインストールする。
- 初回認証はブラウザ操作が必要なので、ユーザーに `colab sessions` の実行を依頼する。

## 標準フロー

依存インストールは必ず通す。`colab run` は依存をインストールできないので使わない。

```sh
colab new -s smc --gpu T4                # GPU は T4 / L4 / A100 から正確に指定（誤記は A100 にフォールバック）
colab install -s smc -r requirements.txt # requirements.txt が空なら省略
colab upload -s smc <local> <remote>     # 自作モジュールを import する場合のみ
colab exec -s smc -f <script>.py         # stdout/stderr がストリームされる
colab download -s smc <remote> <local>   # 結果を回収（stop 前に必ず）
colab stop -s smc                        # 終了時は必ず停止（放置すると課金枠を消費）
```

## 依存ライブラリ

- `requirements.txt` には Colab ランタイムに**入っていない**ライブラリだけを書く。
- torch / transformers / sentence-transformers はランタイム既存の版を使う。上げる必要がある時だけ `colab install -s smc -U <pkg>` → `colab restart-kernel -s smc`。
- ライブラリを追加したら `requirements.txt` を更新してから `colab install` し直す。

## 機械学習モデル実行時の注意

- 再現性のため、torch / transformers / sentence-transformers のバージョンと GPU 名を結果と一緒に出力する。
- VM は使い捨て。HF のモデルキャッシュも毎回消えるので、毎回ダウンロードする前提で書く。
- gated モデルを使う時だけ HF トークンが必要。トークンはリポジトリに置かない。
- T4 は bf16 非対応なので fp16 を使う。OOM 時は batch_size を下げる。

## 禁止・トラブル時

- 対話コマンド（`colab repl`、引数なし `colab ssh`、`colab auth`、`colab drivemount`）は使わない。
- シェルコマンドは `colab ssh -s smc -- '<cmd>'`（例: `nvidia-smi`）。
- `Session not found` / 404 / 401 は `colab sessions` で確認し、`colab new` で作り直す。カーネルが固まったら `colab restart-kernel -s smc`。
- 詳細は `colab help <cmd>`。
