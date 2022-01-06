# push-gem-to-gpr-action

GitHub Package Registry に Gem を Publish する GitHub Action です。  
Release 経由でのトリガーなど、GitHub Ref が `refs/heads/tag` の形になっている場合は記述されている通りのバージョンで Push されます。

それ以外のケース (`workflow_dispatch`による手動実行など) では、以下のような挙動となります。

- Ruby の場合は `0.6.0.627bcfb.pre` のように commit hash と `.pre` が付与されたバージョンで Push されます。
- Java の場合は `build.gradle` 内で以下のように version の出し分けを行うことを想定しています。

```groovy
version = {
    def vd = versionDetails()
    if (vd.commitDistance == 0 && vd.lastTag ==~ /^[0-9]+\.[0-9]+\.[0-9]+(\.[a-zA-Z0-9]+)?/) {
        vd.lastTag
    } else {
        "0.0.0.${vd.gitHash}"
    }
}()
```

## 使い方

### YAML 記述例

```yaml
name: Ruby Gem
​
on:
  workflow_dispatch:
  push:
    tags:
      - 'v*'
​
jobs:
  build:
    name: Build + Publish
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read​
    steps:
      - uses: actions/checkout@v2
      - name: Set up Ruby 2.7
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 2.7
      - name: push gem
        uses: trocco-io/push-gem-to-gpr-action@v1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
```

### inputs

| input        | description                                                                                                                                                                                                                                                 | required | default        |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------------- |
| language     | plugin が書かれている言語を `java` か `ruby`で指定してください。                                                                                                                                                                                                | o        | `ruby`         |
| gem-path     | push 対象の gem が生成される path を指定してください。                                                                                                                                                                                                        | o        | `./pkg/\*.gem` |
| github-token | GPR に push するための GitHub Token を`secrets.GITHUB_TOKEN`で渡して下さい。 <br> 参考: [Automatic token authentication \- GitHub Docs](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token) | o        |                |
