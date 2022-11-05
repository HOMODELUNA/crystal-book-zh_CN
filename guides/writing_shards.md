# 编写 Shard

这篇文章讲述了如何编写 Crystal Shard。

## *什么是 Shard?*
简单来说， Shard 是 Crystal 代码包， 用于共享给其他项目使用。

## 概述

这篇教程中，我们会制作一个叫 _palindrome-example_ 的代码库.

> 给不认识它的人：回文(palindrome)是一种从前到后和从后到前读都一样的词。例如： racecar, kayak, madam，“信言不美，美言不信”，“落花闲院春衫薄，薄衫春院闲花落”

### 要求

为了按照教程发布一个 Crystal Shard，你需要这些东西：
* 一个可用的 [Crystal 编译器](../using_the_compiler/README.md)
* 一个可用的 [Git](https://git-scm.com)
* 一个 [GitHub](https://github.com) 账户

### 创建项目

用[Crystal 编译器](../using_the_compiler/README.md)的 `init lib` 命令，以标准目录结构创建Crystal库.

在终端中输入： `crystal init lib <YOUR-SHARD-NAME>`

例如：
```bash
 $  crystal init lib palindrome-example
      create  palindrome-example/.gitignore
      create  palindrome-example/.editorconfig
      create  palindrome-example/LICENSE
      create  palindrome-example/README.md
      create  palindrome-example/.travis.yml
      create  palindrome-example/shard.yml
      create  palindrome-example/src/palindrome-example.cr
      create  palindrome-example/src/palindrome-example/version.cr
      create  palindrome-example/spec/spec_helper.cr
      create  palindrome-example/spec/palindrome-example_spec.cr
Initialized empty Git repository in /<YOUR-DIRECTORY>/.../palindrome-example/.git/
```

...然后 `cd` 进这个目录：

如：
```bash
cd palindrome-example
```

然后用 `add` & `commit` 把文件加入进Git档案：

```bash
 $  git add -A
 $  git commit -am "First Commit"
[master (root-commit) 77bad84] First Commit
 10 files changed, 102 insertions(+)
 create mode 100644 .editorconfig
 create mode 100644 .gitignore
 create mode 100644 .travis.yml
 create mode 100644 LICENSE
 create mode 100644 README.md
 create mode 100644 shard.yml
 create mode 100644 spec/palindrome-example_spec.cr
 create mode 100644 spec/spec_helper.cr
 create mode 100644 src/palindrome-example.cr
 create mode 100644 src/palindrome-example/version.cr
```

### 编写代码

怎么写代码取决于你自己，但是你写的成果会决定别人是否愿意用你的库，还有你如何维护它。

#### 测试代码
- 测试你的代码，全部。这是让所有人确定它是否如期工作的唯一方法，包括你自己。
- Crystal有 [内置的测试库](https://crystal-lang.org/api/Spec.html)，用它就行了。

#### 文档
- 用规范的注释解释你的代码，全部，即使是私有方法。
- Crystal有 [内置的文档生成器](../conventions/documenting_code.md)，用它就行了。

运行 `crystal docs` 来把你的代码和注释转换为带链接的API文档。你可以用浏览器打开`/docs/` 目录来查看它。

下面讲述了如何把你编译器产生的文档连接到 GitHub Pages。

只要你的文档写完了，并且可以达到，就把这个文档链接加入到你的README.md，这样用户就知道它们。
(记得对应地更改 `<LINK-TO-YOUR-DOCUMENTATION>` )

```Markdown
[![Docs](https://img.shields.io/badge/docs-available-brightgreen.svg)](<LINK-TO-YOUR-DOCUMENTATION>) 
```

### 写 README

README 是你项目的脸面，对项目的成败都有重大影响。
[Awesome README](https://github.com/matiassingers/awesome-readme) 是这方面的一个优秀例子，并且提供了充分的资源。

最重要的，你的 README 应当解释： 
1. 你的库是什么
2. 它干什么
3. 怎么用它

这些解释都应当有良好的结构，辅以充实的样例。

注意： 一定要把Crystal创建的 README 模板里的 `[your-github-name]` 改成你自己的 GitHub 用户名。


#### 代码风格
- 有自己的代码风格是好事，不过遵循 [Crystal 团队指定的核心准则](../conventions/coding_style.md)有助于增强代码一致性，可读性，也便于他人阅读和使用。
- 利用Crystal的 [内置格式化器](../conventions/documenting_code.md) 可以自动格式化某个目录中的所有 `.cr`文件。

例如： 
```
crystal tool format
```

在命令参数中加 `--check` 可以检查你的代码是否已被正确格式化，即格式化起是否没有产生任何影响。

例如： 
```
crystal tool format --check
```

下面的 Travis CI 会教你如何 below to implement this in your build.


### 编写 `shard.yml`

以[spec](https://github.com/crystal-lang/shards/blob/master/SPEC.md#names) 为准。

#### 名称
你的 `shard.yml`的 `name` 属性应当准确地概括你的库。

- 搜索 [crystalshards.xyz](https://crystalshards.xyz/)来检查你的名字是否已经被占用。

例如
```YAML
name: palindrome-example
```

#### 描述
`shard.yml`中的 `description`域是对你Shard的大致描述。

`description` 是用于找到你的Shard的一行概述。

描述应当：
1. 翔实
2. 浅显

#### 优化
如果其他人压根找不到你的项目,那就很难用到它。
[crystalshards.xyz](https://crystalshards.xyz/) 目前是 Crystal 库的索引，我们将对它作出优化。

人们会询问库的*大致*功能和*准确*功能。
比如， Bob 想要一个回文数库，但是 Felipe 想要一个涉及文本的库， Susan 又想要一个拼写检查库。

我们的 `name`域已经是 Bob想要的 "palindrome"，所以我们就不用重复 _palindrome_ 关键词。另外，我们要满足 Susan搜索的 "spelling"和 Felipe搜索的 "text"。
```YAML
description: |
  A textual algorithm to tell if a word is spelled the same way forwards as it is backwards.
```

### GitHub

- 创建一个仓， `name`和 `description`都和你的 `shard.yml`相同。

- 加入所需的代码
```bash
$ git add -A && git commit -am "shard complete"
```
- 设置远程仓: (记得对应地修改 `<YOUR-GITHUB-USERNAME>`和 `<YOUR-REPOSITORY-NAME>`)

注意： 你可以随意把 `public`替换为 `origin`，或是其他你喜欢的名字。
```bash 
$ git remote add public https://github.com/<YOUR-GITHUB-NAME>/<YOUR-REPOSITORY-NAME>.git
```
- Push 上去： 
```bash
$ git push public master
```

#### GitHub Releases
GitHub Releases 是个好的实践。

在你的README中加入编译提示，来提醒用户 release 的位置：
 (记得对应地修改 `<YOUR-GITHUB-USERNAME>`和 `<YOUR-REPOSITORY-NAME>`)

```Markdown
[![GitHub release](https://img.shields.io/github/release/<YOUR-GITHUB-USERNAME>/<YOUR-REPOSITORY-NAME>.svg)](https://github.com/<YOUR-GITHUB-USERNAME>/<YOUR-REPOSITORY-NAME>/releases)
```

一开始线把它设为你的 _releases_ 页面。
  - 它们在 `https://github.com/<YOUR-GITHUB-NAME>/<YOUR-REPOSITORY-NAME>/releases`

点击 "Create a new release".

根据 [ Crystal Shards README](https://github.com/crystal-lang/shards/blob/master/README.md), 
> 当库在 repositories发布时, 这个仓应当有 semver式的版本标签，以 `v`为前缀。例如： v1.2.3, v2.0.0-rc1 或 v2017.04.1

相应地，在 `tag version`输入你的版本, 比如 `v0.1.0`，这一定要与  `shard.yml` 中的`version`匹配。写上 `v0.1.0`，然后写一小段发布通告。 

点击 "Publish release"，完成。

你现在就会注意到README 里面的 GitHub Release badge已经更新了。

遵循[语义版本](http://semver.org/)，每次创建新Realease时都把你的代码推送到 `master`。

### Travis CI 和 `.travis.yml`
如果你没有准备好，就先去 [注册 Travis CI 账号](https://travis-ci.org/)。

在你README.md 的description 下方加入 build badge:
 (记得对应地修改 `<YOUR-GITHUB-USERNAME>`和 `<YOUR-REPOSITORY-NAME>`)
```Markdown
[![Build Status](https://travis-ci.org/<YOUR-GITHUB-USERNAME>/<YOUR-REPOSITORY-NAME>.svg?branch=master)](https://travis-ci.org/<YOUR-GITHUB-USERNAME>/<YOUR-REPOSITORY-NAME>) 
```
Build badges 可以简洁地告诉人们你的代码是否通过了 Travis CI 编译。

把这些加入你的 `.travis.yml`：
```YAML
script:
  - crystal spec
```

这会让 Travis CI运行你的测试样例。 
根据命令的结果， Travis CI 会返回一个 [构建状态](https://docs.travis-ci.com/user/customizing-the-build/#Breaking-the-Build) ，它是 "passed", "errored", "failed" 或 "canceled" 的一种。


如果你希望所有的代码都以 `crystal tool format`格式化，增加一行 `crystal tool format --check`。如果代码没有正确格式化，这就会 [破坏构建状态](https://docs.travis-ci.com/user/for-beginners/#Breaking-the-Build) ，就像测试失败了一样。

例如
```YAML
script:
  - crystal spec
  - crystal tool format --check
```


Commit ，然后 push 到 GitHub。

按照[如下指示](https://docs.travis-ci.com/user/getting-started/) 建立 repo & 运行 Travis CI。

一旦你运行起来，编译通过，你README中的 build badge就会被更新。


#### 把 `docs` 绑定到 GitHub-Pages

把这些 `script` 加入你的 `.travis.yml`:
```YAML
  - crystal docs
```

这会让 Travis CI 创建你的文档。

然后，把这些部分加入你的 `.travis.yml`：
(记得对应地更改 `<YOUR-GITHUB-REPOSITORY-NAME>`)
```YAML
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  project_name: <YOUR-GITHUB-REPOSITORY-NAME>
  on:
    branch: master
  local_dir: docs
```

用你的[personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)和 `GITHUB_TOKEN`来[设置环境变量](https://docs.travis-ci.com/user/environment-variables#Defining-Variables-in-Repository-Settings)。

一路下来 `.travis.yml`应当看着像这样：

```YAML
language: crystal
script:
  - crystal spec
  - crystal docs
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  project_name: <YOUR-GITHUB-REPOSITORY-NAME>
  on:
    branch: master
  local_dir: docs
```

[这里](https://docs.travis-ci.com/user/deployment/pages/) 有用 Travis CI 部署 GitHub-Pages 的官方文档。
