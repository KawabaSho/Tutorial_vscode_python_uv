# Github と VS Code ではじめる Python 環境

プロジェクト管理を高速に処理できる `uv` を使ってPython環境を整え、ソースコードを Github 上の 非公開 Repository で管理していく方法を説明します。

## 前提条件
- Windows 環境
- VS Code インストール済み
- GitHub アカウント作成済み
- Python が PowerShell 上で使用可能

## uv のインストール方法
簡単な導入方法を説明します（推奨のインストール方法は、https://docs.astral.sh/uv/getting-started/installation/ を確認してください）。PowerShell で以下のコマンドを実行します。バージョン情報が出てきたら成功です。
```powershell
pip install uv
uv --version
```

## プロジェクトの作成
プロジェクトの作成と`uv`による初期化を以下のコマンドで実行します。プロジェクトを作りたいフォルダに移動して、右クリックで PowerShell を開きます。
```powershell
mkdir MyProject
cd MyProject
uv init
```

または

```powershell
uv init MyProject
cd MyProject
```

すでにフォルダが存在する場合はフォルダ内で以下を実行します。

```powershell
uv init 
```

これで `main.py`、`pyproject.toml`、`.python-version`、`README.md` などがそろい、Python プロジェクトの土台ができます。必要なライブラリは `uv add` で追加し、実行は `uv run` で行います。以下のコマンドは`requests`と`numpy`の追加、削除およびプログラムの実行方法です。

```powershell
uv add requests numpy
uv remove requests
uv run main.py
```

次に、フォルダを VS Code で開きます。

```powershell
code .
```

#### uv 環境で完結させる方法
仮想環境 `.venv` が存在する場合、VS Code 起動すると仮想環境を自動でアクティベートしてしまいます。その結果、uv 機能を使わず、PC にある Python.exe が 仮想環境 `.venv` でプログラムを実行できるようになります。

```powershell
python main.py
```

特に問題ありませんが、uv 環境でプログラムの実行も統一した方がパッケージ管理も安全で高速です。以下では、 VS Code 起動時に仮想環境が起動しないようにし、`ctrl + shift + B`で `uv run main.py` が実行されるようにします。不要な方は読み飛ばしてください。

#### - 自動アクティベートの無効化とビルドショートカットの作成

`.vscode` フォルダを作成し、その中に `settings.json` と `tasks.json` を作成します。 `settings.json` 内は

```settings.json
{
    "python.defaultInterpreterPath": "${workspaceFolder}\\.venv\\Scripts\\python.exe",
    "python.terminal.activateEnvironment": false
}
```
`tasks.json`内は
```tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Run Current Python File (uv)",
            "type": "shell",
            "command": "uv",
            "args": [
                "run",
                "${file}"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "presentation": {
                "reveal": "always",
                "panel": "shared",
                "focus": false,
                "clear": true
            },
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": []
        }
    ]
}
```
とします。`main.py` を開き、 `ctrl + shift + B` を押せば、ターミナルが表示され実行結果が表示されます。どこのファイルを開いた状態でも `main.py` を実行したい場合は、`tasks.json`内の "arg" を以下のように書き換えてください。

```tasks.json
"args": [
    "run",
    "main.py"
],
```
以上で uv を使った開発環境が整いました。次は、Github 上で管理する方法を説明していきます。ローカルなままで開発する場合は以降のチュートリアルは不要です。

## SSH キーの設定
ローカル環境で作成したソースコードを GitHub にアップロード( Github では Push といいます)するには、GitHub への認証が必要になります。SSH を使う場合は公開鍵を GitHub に登録し、HTTPS を使う場合は Personal Access Token などで認証します。複数人で Push する場合は、各メンバーを collaborator として追加するか、organization / team で write 権限を付与し、各自が自分の SSH キーまたは Personal Access Token で認証します。

#### - SSH キーの作成
VS Code 上の Terminal で以下のコマンドを実行してみてください。`C:\Users\ユーザー名\.ssh` に `id_rsa.pub` という公開鍵が生成されます。

```powershell
ssh-keygen -t rsa
Get-ChildItem $env:USERPROFILE\.ssh
```

生成時に名前を付けたい場合や公開鍵のファイル内にPCの名前が入るのが嫌な方は

```
ssh-keygen -t rsa -C "(ユーザー)@gmail.com" -f $env:USERPROFILE\.ssh\(公開鍵名)
```

などにしてください。公開鍵が作成出来たら、`C:\Users\ユーザー名\.ssh\(公開鍵名).pub` を VS Code で開き、中身をすべてコピーしてください。

#### - SSH キーの登録

ここで、Github 上で、`アカウント -> Setting -> SSH and GPG keys` に移動し、`New SSH key`を押してください。`Title` は自由に決めて、`Key` の中にコピーした公開鍵を張り付けてください。VS Code の Terminal に移動して、

```powershell
ssh -T git@github.com
```

で成功すれば、Github に SSH の登録は完了です。次に Github 上で新しい非公開の repository を作成します。ちなみに、`gh repo create YOUR_REPOSITORY --private --source . --remote origin --push`などでローカル環境から Github 上に新規 repository を作る方法もあるようです。

## Github 側の作業
Github 上で非公開の Repository を新規作成してください。作成出来たら、`git@github.com:ユーザー名/リポジトリ名.git`が表示されると思います。以上で、Github 上の設定は完了です。

## プロジェクト内で Github のアカウントの登録と Repository の連携
いよいよ Github にローカルで作成した uv 環境のアップロード( Push )を行っていきます。まずは、Github のアカウントをプロジェクトのローカル環境に登録していきます。

VS Code 内で Termial を開いて、作成したプロジェクト内で以下のコマンドを実行してください。ただし、Github 上で、`アカウント -> Setting -> Emails`内の `Keep my email addresses private` と `Block command line pushes that expose my email` を有効にしている場合、Git の `user.email` に個人メールアドレスが入っていると push が拒否されることがあります。非公開の Repository を使う場合、GitHub の `noreply` メールアドレスを確認し、それを Git に設定しておくのが安全なようです。`noreply` メールアドレスは、`アカウント -> Setting -> Emails` 内に `(数字 + ユーザー名)@users.noreply.github.com ` といった感じでこっそり書いてあります。

```powershell
git init               
git config --local user.name "ユーザー名"                   
git config --local user.email "Github上のメールアドレス"         
```

最後に、uv 環境が整ったローカルの内容を GitHub に Push します。

```powershell
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin git@github.com:ユーザー名/リポジトリ名.git
git push -u origin main
```

ここまでできれば、VS Code で編集し、`uv` で実行し、GitHub の非公開の repository で履歴を安全に管理する環境ができたことになります。以降は `uv add`、`uv run`、`git add`、`git commit`、`git push` を繰り返すだけで、学習用にも個人開発用にも十分に運用できます。

なお、コマンドで Repository への Pull / Push が面倒な場合は、VS Code 内で GitHub の拡張機能を利用して、GUIベースで管理ができます。


## References

- [uv: Introduction](https://docs.astral.sh/uv/)
- [VSCodeで始めるGit(Hub)管理](https://zenn.dev/kd_gamegikenblg/articles/b220e23b0b7ef9)
