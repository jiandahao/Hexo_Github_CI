# Hexo_Github_CI
利用Github Page和Hexo搭建个人静态博客网站，并通过github action实现自动化部署。

首先需要需要安装相关环境，包括git、nodejs，hexo，这个网上有很多教程。

首先在github上创建一个名为<your_github_username>.github.io的repo，并创建一个叫做`hexo`（这个自定义），拉取仓库到本地，切换到`hexo` 分支。

创建一个hexo工程，在一个空目录下执行`hexo init`即可。完成创建后的工程目录为
```bash
.
├── _config.yml
├── node_modules
├── package.json
├── scaffolds
├── source
└── themes
```
然后把以上内容全部移动到<your_github_username>.github.io仓库的`hexo`分支中，提包保存即可。在github上创建action，实现一个CI
```yaml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. Triggers the workflow on push or pull request 
# events but only for the master branch
on:
  push:
    branches: [ hexo ]
  pull_request:
    branches: [ hexo ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2

    # Runs a single command using the runners shell
    - name: deploy
      run: |
        npm install hexo-cli
        export HEXO_PATH=${PWD}/node_modules/hexo-cli/bin
        export PATH=${PATH}:${HEXO_PATH}
        npm install
        npm install --save hexo-deployer-git
        git config --global user.name jiandahao
        git config --global user.email jiandahao@gmail.com
        export TOKEN="${{ secrets.GH_TOKEN }}"
        sed -i "s/GH_TOKEN/${TOKEN}/g" ./_config.yml
        hexo generate && hexo deploy
```
这里需要注意的是，我们需要先创建一个access token，这样我们才可以在CI环境中自动通过github的验证，平时我们都是通过SSH或者输入帐号密码的方式完成push的，但是在该CI环境中显然不行，所有我们可以使用github提供的access token方式。在[这里](https://github.com/settings/tokens)创建你的token。然后再创建一个secret key，在仓库的settings里面。

接下来我们创建一个专门用来保存我们平时的笔记，文章等的仓库。假设仓库名为`notes`，
创建完成后给该笔记仓库也添加一个CI
```yaml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. Triggers the workflow on push or pull request 
# events but only for the master branch
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2

    # Runs a single command using the runners shell
    - name: deploy to hexo blog
      run: |
        git config --global user.name <your username>
        git config --global user.email <your email>
        cd ..
        export TOKEN="${{ secrets.GH_TOKEN }}"
        git clone -b hexo --depth 1 https://${TOKEN}@github.com/BugSurvivor/BugSurvivor.github.io.git
        cp -a notes BugSurvivor.github.io/source/_posts
        rm -rf BugSurvivor.github.io/source/_posts/notes/.git
        rm -rf BugSurvivor.github.io/source/_posts/notes/.github
        cd BugSurvivor.github.io
        git add --all && git commit -m"update notes" && git push origin hexo
```
  
通过以上的设置，即可实现每次我们往`notes`仓库push新的笔记时，就会自动更新<your_github_username>.github.io仓库hexo分支的内容，并完成hexo博客网站的自动化部署啦
