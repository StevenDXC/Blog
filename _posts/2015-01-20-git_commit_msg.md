---
layout:     post
title:      "git提交信息规范"
subtitle:   ""
date:       2015-01-20 13:00:00
author:     "steven"
catalog:    true
tags:
    - git
---

使用Git时，每次提交代码，都要写 Commit message（提交说明），否则git就不允许提交本次改动。

```java
$ git commit -m "msg"
```

msg就是提交说明，默认个情况下写什么都行。但是一般情况下，应该简述下本次改动的内容和影响的范围。
目前有多种填写规范，但是使用最广的还是[Angular规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)。

 Commit Message 的格式
 ---

包括三部分，Header,Body和Footer

```java
<type>(<scope>): <subject>
// 换行
<body>
// 换行
<footer>
```

Header 是必需的，Body和Footer可以省略。尽量不要让每行超过72个字符，避免显示时换行影响美观。

Header
----

Header包括三个字段：type（必需）、scope（可选）和subject（必需）

type用来说明本次提交的类型，只能使用以下7种中的一种：

```java
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
```

scope用来说明本次改动的影响范围，如业务模块，类，界面等

subject用来简单说明下本次改动，最好不要超过50个字。尽量以第一人称动词开头，结尾不加。好。第一个单词的首字母小写。


Body
---

Body 是对本次 commit 的详细描述，可以分成多行，如：

```java
.Fix small typo in docs widget (tutorial instructions)
.Fix test for scenario.Application - should remove old iframe
.docs - various doc fixes
.docs - stripping extra new lines
.Replaced double line break with single when text is fetched from Google
.Added support for properties in documentation
```

body也是用第一人称来书写，应该说明本次提交的动机，以及和之前的对比


Footer
---

footer只有两种情况需要书写

1.不兼容的改动

若本次改动与上次改动不兼容，则footer应该以BREAKING CHANGE开头，后面加上变动的理由，描述和迁移的方法。如：

```java
Before:

   scope: {
     myAttr: 'attribute',
     myBind: 'bind',
     myExpression: 'expression',
     myEval: 'evaluate',
     myAccessor: 'accessor'
   }

   After:

   scope: {
     myAttr: '@',
     myBind: '@',
     myExpression: '&',
     // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
     myAccessor: '=' // in directive's template change myAccessor() to myAccessor
   }
```

2.关闭 Issue

若本次改动关闭了某个或多个Issue，那应该在footer中加以描述。如：

```java
Closes #123, #245, #992
```

特殊状况：

1.Revert

如果当前 commit 用于撤销以前的 commit，则应该以revert:开头，后面跟着被撤销 Commit 的 Header。如;

```java
revert: feat(view): add 'width' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.

```

2.merge branch 和 修改冲突，git 会自动生成 commit msg.忽略就行了


验证
---

可以使用[git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)来验证commit msg。每次提交的时候都会触发commit-msg的git hook.只需要在.git/hooks下的commit-msg文件中编写校验规则的shell脚本。提交代码的时候会自动执行改脚本，校验commit msg是否符合规范。

git hookhai还还支持python,python支持正则表达式，所以在mac或Linux可以直接使用python和正则表达式来验证commit msg。在windows上也可以，但是需要在windows上先安装python。

>validate-commit-msg

[validate-commit-msg](https://github.com/kentcdodds/validate-commit-msg)是一个用来验证commit msg是否合规的脚步，用JS编写。需要配合[ghooks](https://www.npmjs.com/package/ghooks)使用。比较适合WEB前端开发。具体使用方法参看对应的说明。
