# hugo 自动部署项目

1. 在 hugo 主目录下新建 .github/workflows/hugo.yaml 文件，复制以下内容

~~~ yaml
name: deploy

# 代码提交到main分支时触发github action
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
        - name: Checkout
          uses: actions/checkout@v4
          with:
              fetch-depth: 0

        - name: Setup Hugo
          uses: peaceiris/actions-hugo@v3
          with:
              hugo-version: "latest"
              extended: true

        - name: Build Web
          run: hugo -D

        - name: Deploy Web
          uses: peaceiris/actions-gh-pages@v4
          with:
              PERSONAL_TOKEN: ${{ secrets.TOKEN }}
              EXTERNAL_REPOSITORY: 你的github名/你的仓库名
              PUBLISH_BRANCH: main
              PUBLISH_DIR: ./public
              commit_message: auto deploy

~~~

2. 在 github settings -> Developer Settings -> Personal access tokens -> Tokens (classic) 中新建一个 token，Expiration 设置为 **No expiration**，勾选**repo** 和 **workflow**，得到一个 token
3. 新建一个 github 仓库，在新仓库中 Settings -> Secrets and variables -> Actions -> New repository secret 新建一个仓库密钥，设置 Name 为 **Token**，Secret 为之前获取的 token
4. 把 hugo 主目录连接到当前 github 仓库并提交，以后博客进行修改，都可以在当前目录下提交

