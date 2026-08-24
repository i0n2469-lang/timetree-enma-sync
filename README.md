# TimeTree公開カレンダー「enma」→ Googleカレンダー 自動同期

TimeTreeの公開カレンダー

https://timetreeapp.com/public_calendars/enma

を3時間おきに確認して `enma.ics` に変換し、GoogleカレンダーからURL購読できるようにする仕組みです。

**TimeTreeへのログイン情報やGoogleアカウント情報は使いません。**
公開されているTimeTreeカレンダーだけを取得します。

---

## 何が起きるの？

```text
TimeTree公開カレンダー「enma」
        ↓
GitHub Actionsが約3時間おきに確認
        ↓
public/enma.ics を自動更新
        ↓
GitHub Pagesで固定URLとして公開
        ↓
GoogleカレンダーがそのURLを購読
```

一度設定すれば、基本的には放置でOKです。

※ GitHub側は3時間おきに確認しますが、Googleカレンダーが購読URLを再取得するタイミングはGoogle側に任されます。そのため、TimeTreeで予定が更新されてもGoogleカレンダーへの反映にはさらに数時間かかる場合があります。当日の急な予定変更を拾う用途には向きません。

---

# 最初の設定

## 0. 先にこのzipを展開する

このファイル一式をWindows上の適当な場所に展開してください。

中には最低でも次の2つがあります。

```text
README.md
.github/
  └ workflows/
      └ update-ics.yml
```

`.github` フォルダも必要です。消さないでください。

---

## 1. GitHubアカウントを作る

GitHubを使ったことがなければ、まず

https://github.com/

でアカウントを作ります。

GitHubのユーザー名は、あとでICSの公開URLにも入ります。

---

## 2. 新しいリポジトリを作る

GitHubにログインしたら、右上の `+` → **New repository** を選びます。

設定例：

- **Repository name**: `timetree-enma-sync`
- **Description**: 空欄でOK
- **Public** を選択
- `Add a README file` はOFFのまま
- `.gitignore` は None
- License は None

最後に **Create repository** を押します。

### なぜPublic？

GitHub Pagesを一番簡単に無料で使うためです。

ここに置くのは、もともと公開されているTimeTreeの予定と、この同期用設定だけです。TimeTreeやGoogleのパスワード・ログイン情報は置きません。

---

## 3. このフォルダの中身をGitHubへアップロードする

作ったばかりのリポジトリ画面で、

**uploading an existing file**

を押します。

Windowsで展開したフォルダを開き、**フォルダそのものではなく中身**をアップロードします。

必要なのは：

```text
README.md
.github フォルダ
```

です。

アップロード画面の下にある **Commit changes** を押します。

アップロード後、GitHub上で

```text
.github/workflows/update-ics.yml
```

が見えればOKです。

---

## 4. GitHub Actionsに書き込みを許可する

上のタブから

**Settings** → 左側の **Actions** → **General**

へ進みます。

下の方にある **Workflow permissions** で

**Read and write permissions**

を選び、**Save** を押します。

これで自動処理が `public/enma.ics` をリポジトリへ保存できるようになります。

---

## 5. 一度だけ手動実行する

上のタブから **Actions** を開きます。

左側に

**Update TimeTree ICS**

が出ていたら選択します。

右側の **Run workflow** → もう一度 **Run workflow** を押します。

しばらくすると実行履歴が表示されます。

- 緑のチェック → 成功
- 赤い× → 失敗

成功すると、リポジトリのトップに

```text
public/
  └ enma.ics
```

が追加されます。

**ここまで成功すれば、TimeTreeから予定を取る部分は完成です。**

---

## 6. GitHub PagesをONにする

上のタブから

**Settings** → 左側の **Pages**

へ進みます。

**Build and deployment** で：

- **Source**: `Deploy from a branch`
- **Branch**: `main`
- フォルダ: `/ (root)`

を選んで **Save** を押します。

少しするとGitHub Pagesの公開URLが作られます。

Repository nameを `timetree-enma-sync` にした場合、ICSのURLはだいたい次の形です。

```text
https://あなたのGitHubユーザー名.github.io/timetree-enma-sync/public/enma.ics
```

ブラウザでこのURLを開いて、ファイルが表示またはダウンロードされればOKです。

---

## 7. Googleカレンダーへ登録する

これはPCブラウザ版のGoogleカレンダーから行います。

Googleカレンダーを開いて、左側の

**他のカレンダー** の `+`

→ **URLで追加**

を選択します。

そこへ先ほどの

```text
https://あなたのGitHubユーザー名.github.io/timetree-enma-sync/public/enma.ics
```

を貼り付けて、**カレンダーを追加** を押します。

これで登録完了です。

スマホでも、同じGoogleアカウントを使っていれば表示されます。

---

# 自動更新について

## GitHub側

通常は3時間おきにTimeTreeを確認します。

実行時刻はUTCで：

```yaml
- cron: '17 */3 * * *'
```

です。

毎時ちょうど（0分）はGitHub Actionsが混雑しやすいため、17分にずらしています。

予定に変更がなければ、余計な更新コミットは作りません。

## 60日停止対策

GitHubのPublicリポジトリでは、60日間リポジトリに活動がないとscheduled workflowが自動停止することがあります。

その対策として、この設定では**月1回だけ、予定に変更がなくても小さな空コミットを自動作成**します。

普段は意識しなくて大丈夫です。

## Google側

Googleカレンダーが外部ICSを再取得する間隔は、こちらから指定できません。

そのため、

```text
TimeTree更新
→ 最大約3時間でGitHub側ICS更新
→ その後Googleが再取得した時点で反映
```

という動きになります。

ライブやイベントなど、ある程度前もって登録される予定の同期には向いています。

---

# この仕組みに入っているもの

- TimeTree公開カレンダーの取得
- ICS形式への変換
- 3時間おきの自動更新
- GitHub Pagesでの固定URL公開
- GitHub Actionsの60日自動停止対策
- 手動実行ボタン

`timetree-exporter` は公開カレンダー対応版の **0.8.0** に固定しています。

将来、新版が出ても勝手に切り替わらないため、急に挙動が変わりにくい構成です。

---

# 入らない・完全には再現できない可能性がある情報

TimeTree公開APIの仕様により、予定によっては次の情報が完全には取得できない場合があります。

- TimeTree独自の通知・アラート
- 一部のラベル情報
- 一部の繰り返し情報
- TimeTree固有の表示情報

予定名、開始・終了日時、終日予定、説明などの基本情報はICSとして出力されます。

また、Googleカレンダー側でTimeTreeと同じ色分けが再現されるとは限りません。

---

# うまくいかない時

まず **Actions** → **Update TimeTree ICS** を開いて、最新の実行結果を見ます。

赤い×になっていたら、その実行をクリックするとエラー内容が表示されます。

その画面をスクリーンショットするか、エラー文をコピーして確認すれば原因を追えます。

特に最初は、次の順番で確認すると分かりやすいです。

1. `.github/workflows/update-ics.yml` がGitHub上に存在するか
2. Actionsの手動実行が緑のチェックになるか
3. `public/enma.ics` が作られたか
4. GitHub PagesのICS URLがブラウザで開けるか
5. そのURLをGoogleカレンダーへ登録できるか

---

# 自動更新なしで一度だけ試す場合

PCにPythonがある場合は、ローカルでも取得できます。

```bash
pip install timetree-exporter==0.8.0
timetree-exporter --public-calendar -c enma -o enma.ics
```

生成された `enma.ics` をGoogleカレンダーの「インポート」から取り込めます。

ただし、こちらは一度取り込むだけなので、その後TimeTreeで予定が変更されても自動更新されません。
