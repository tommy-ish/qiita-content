---
title: GitHub Packagesでnpmパッケージを扱うための設定方法
tags:
  - npm
  - GitHubPackages
private: false
updated_at: '2025-05-11T14:25:35+09:00'
id: ea2f0de0d0eb192620eb
organization_url_name: null
slide: false
ignorePublish: false
---
# GitHub Packagesで公開されているnpmパッケージをインストールする方法

GitHub Packagesで公開されているnpmパッケージをインストールするにはGitHubへ認証する必要があります。GitHubへ認証するにはpersonal access token（classic）が必要です。また、personal access token（classic）にはread:packagesスコープが設定されている必要があります。personal access token（classic）がない場合は発行してください。[New personal access token（classic）](https://github.com/settings/tokens/new)で発行できます。次に

```ini:~/.npmrc
//npm.pkg.github.com/:_authToken=TOKEN
```

のようなファイルを作成してください。`TOKEN`をread:packagesスコープが設定されているトークンに置き換えてください。次にリポジトリのルートのpackage.jsonと同じディレクトリに

```ini:.npmrc
@NAMESPACE:registry=https://npm.pkg.github.com
```

を作成してください。`@NAMESPACE`はインストールしたいパッケージのスコープに置き換えてください。例えば、@tommy-ish/fooというパッケージをインストールしたい場合は`@tommy-ish:registry=http://npm.pkg.github.com`にしてください。後は通常通りインストールできます。例えば、@tommy-ish/fooというパッケージをインストールしたい場合は

```console
$ npm install @tommy-ish/foo
```

を実行します。

# GitHub ActionsからGitHub Packagesにアクセスする方法

GitHub ActionsからGitHub Packagesにアクセスするにはactions/setup-nodeのregistry-urlとNODE_AUTH_TOKENという環境変数を使うと簡単です。具体的には

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22.x
      registry-url: https://npm.pkg.github.com
  - run: npm ci
    env:
      NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

のように書くとGitHub Packagesからパッケージをインストールできます。また

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22.x
      registry-url: https://npm.pkg.github.com
  - run: npm ci
  - run: npm publish
    env:
      NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

のように書くとGitHub Packagesでパッケージを公開できます。

# DependabotがGitHub Packagesで公開されているパッケージをアップデートできるようにする方法

Dependabotの設定ファイルを

```yaml:.github/dependabot.yml
version: 2
registries:
  npm-github:
    type: npm-registry
    url: https://npm.pkg.github.com
    token: ${{secrets.MY_ARTIFACTORY_PASSWORD}}
    replaces-base: true
updates:
  - package-ecosystem: npm
    registries:
      - npm-github
```

のように設定してください。`MY_ARTIFACTORY_PASSWORD`はDependabotシークレットの名前に置き換えてください。そのDependabotシークレットはread:packagesスコープが設定されているトークンが設定されている必要があります。
