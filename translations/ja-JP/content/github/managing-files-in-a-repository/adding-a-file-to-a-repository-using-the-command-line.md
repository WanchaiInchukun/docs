---
title: コマンドラインを使用してファイルをリポジトリに追加する
intro: 'コマンドラインを使って、既存のファイルを {{ site.data.variables.product.product_name }}のリポジトリにアップロードできます。'
redirect_from:
  - /articles/adding-a-file-to-a-repository-from-the-command-line/
  - /articles/adding-a-file-to-a-repository-using-the-command-line
versions:
  free-pro-team: '*'
  enterprise-server: '*'
---

{% tip %}

**ヒント:** [既存のファイルを {{ site.data.variables.product.product_name }} Web サイトから追加](/articles/adding-a-file-to-a-repository)することもできます。

{% endtip %}

{{ site.data.reusables.command_line.manipulating_file_prereqs }}

{{ site.data.reusables.repositories.sensitive-info-warning }}

1. 自分のコンピュータ上で、{{ site.data.variables.product.product_name }}にアップロードしたいファイルを、リポジトリをクローンした際に作成したローカルディレクトリに移動します。
{{ site.data.reusables.command_line.open_the_multi_os_terminal }}
{{ site.data.reusables.command_line.switching_directories_procedural }}
{{ site.data.reusables.git.stage_for_commit }}
  ```shell
  $ git add .
  # ファイルをローカルリポジトリに追加し、コミットするためにステージします。 {{ site.data.reusables.git.unstage-codeblock }}
  ```
{{ site.data.reusables.git.commit-file }}
  ```shell
  $ git commit -m "Add existing file"
  # 追跡された変更をコミットし、リモートリポジトリへのプッシュに備えます。 {{ site.data.reusables.git.reset-head-to-previous-commit-codeblock }}
  ```
{{ site.data.reusables.git.git-push }}

### 参考リンク

- [新しいファイルの作成](/articles/creating-new-files)
- [コマンドラインを使った既存のプロジェクトの GitHub への追加](/articles/adding-an-existing-project-to-github-using-the-command-line)