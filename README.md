# devcontainer trial
コンテナとIDEを統合する機能．

# 事前に必要なこと
- Dockerのインストール
- Dev Containers拡張機能のインストール

# devcontainerを使ったプロジェクトの作成
`.devcontainer/devcontainer.json`を作成する.
[マニュアル](https://containers.dev/implementors/json_reference/)

 このファイルを作成することで自動的にコンテナを利用するプロジェクトとして検出される．
 
 すでに開いているプロジェクトをコンテナで開きなおす場合， Ctrl+P -> Reopen in Containerで開ける．

# devcontainer.jsonの書き方

devcontainer.jsonのみの場合，Dockerfileを使う場合，Docker Composeを使う場合で書き方が少し変わる．

## 共通したdevcontainer.jsonのプロパティ
```jsonc
{
    // コンテナの名前．なんでもよいが他と競合はダメ．
    // ここでは定義済みの変数を利用して，プロジェクトのルートフォルダと同じ名前としている．
    "name": "${localWorkspaceFolderBasename}", 
    //マウントするボリュームのリスト．Docker CLIの--mountフラグと同じ内容．
    "mounts": [
        // node_modulesなどのアクセスが多いフォルダはvolume mountにすると高速になる．
        {
            "type": "volume",
            "source": "${localWorkspaceFolderBasename}_tmp",
            "target": "/${localWorkspaceFolderBasename}/workspace/node_modules"
        }
    ],
    // コンテナに外部から接続するためのポート
    // これを設定することでコンテナ内でサーバを立ち上げて外から触ったりできる．
    // この場合，localhost:8080にアクセスすればコンテナ内のサーバに触れる．
    // コンテナ内のポートは特に指定しなくても，VS Code側でうまく合わせてくれる．
    "forwardPorts": [
        8080
    ],
    // コンテナ起動後に実行されるコマンド
    "postStartCommand": "npm install -g npm",
    // コンテナ作成後に一度実行されるコマンド.
    // ここに初期化とかは余り書かずに，Dockerfileを利用したり，
    // 事前にimageを作成したほうがよさそう．
    "postCreateCommand": "apt update -y && apt upgrade -y && apt install git -y"
}
```
このほかにも設定項目はあるけど使ったことはない．


## devcontainer.jsonのみの場合

```jsonc
{

    "name": "${localWorkspaceFolderBasename}", 
    "image": "node:slim", //利用するイメージの名前
    // Docker CLIに渡す引数のリスト
    "runArgs": [
        "--workdir",
        "/${localWorkspaceFolderBasename}/workspace",
        "--tty",
        "--name",
        "${localWorkspaceFolderBasename}"
    ],
    "mounts": [
        {
            "type": "volume",
            "source": "${localWorkspaceFolderBasename}_tmp",
            "target": "/${localWorkspaceFolderBasename}/workspace/node_modules"
        }
    ],
    // コンテナを開いたときにマウントされるボリュームを指定する． 
    // このオプションをセットする場合workspaceFolderもセットされている必要がある．
    "workspaceMount": "source=${localWorkspaceFolder},target=/${localWorkspaceFolderBasename},type=bind,consistency=cached",
    // このプロジェクトをコンテナで開いたときに，エディターなどが開くフォルダーのパスを指定する．
    "workspaceFolder": "/${localWorkspaceFolderBasename}/workspace",
    "forwardPorts": [
        8080
    ],
    "postStartCommand": "npm install -g npm",
    "postCreateCommand": "apt update -y && apt upgrade -y && apt install git -y"
}
```

`devcontainer.json`をgitの管理に含みたい場合，`.git`フォルダが含まれるフォルダもマウントするとよい．

```text
project/
    ├─ .git/
    ├─ .devcontainer/
    │   └─ devcontainer.json
    └─ workspace/
        └─ プロジェクトの中身
```
`workspaceMount`に`project/`を指定してマウントし，
`workspaceFolder`に`workspace/`フォルダを指定することで，`.git`を含みつつ，`workspace/`をコンテナで開いたように扱える．

この場合の`devcontainer.json`は
```json
{
    "workspaceMount": "source=absolute/path/to/project,target=/project,type=bind,consistency=cached",
    "workspaceFolder": "/project/workspace",
}
```
とすればよい．この際，定義済みの変数`${localWorkspaceFolderBasename}`が`project`になるため，先に提示した`devcontainer.json`ではこの変数を利用することで汎用的にできる．

## Dockerfileを使う場合
TODO
## Docker Composeを使う場合
TODO