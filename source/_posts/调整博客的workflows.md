---
title: 调整博客的workflows
date: 2024-06-15 11:48:46
tags: [hexo]
---

由于调整了域名，我的 blog 需要更新一下域名的配置。具体来说就是使用了 github 的 Custom Domain 功能，需要重新改一下 CNAME 中的内容。

因为我弄了 Github Action ，所以推送代码之后会自动部署。但是却发布失败了。

<!-- more -->

## 发生了什么？

看了一下 Github Action 的日志，错误发生在 `sma11black/hexo-action@v1.0.4` 这个 action 上，可能是由于 node 版本太低。

然后我又看了一下这个 action 的仓库，发现它是 docker 编译，使用的 `node 12` 的镜像，翻了一下 issue 好像没有人提到这个问题。

只能自己动手解决了。

## 解决方法

删掉这个 action ，直接使用 node 和 pnpm 的 `setup action`，加上脚本即可解决。

我发布 `Github Pages` 使用的是 deploy 的 git 插件，所以需要提供一个密钥，这个密钥需要设置在环境变量中。

我没有使用常规的创建目录、写入 id_rsa 的操作，而是使用 ssh-agent 的方式提供 git 的认证。

所以我的配置如下（部署的配置）

```yaml
- name: Deploy
  shell: bash
  run: |
    eval "$(ssh-agent -s)"
    echo "${{ secrets.DEPLOY_KEY }}" | ssh-add -
    git config --global init.defaultBranch master
    git config --global user.name xxxxxxxxxxxx
    git config --global user.email xxxxxxxxxxxx
    pnpm run deploy -g
```

完整的配置可以看 [deploy.yml](https://github.com/weilence/weilence.github.io/blob/master/.github/workflows/deploy.yml)