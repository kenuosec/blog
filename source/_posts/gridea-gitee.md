---
title: Gridea 魔改使其支持 Gitee Pages
cover: https://pic.rmb.bdstatic.com/bjh/0d9eb92f67ba93b514f2555a70eacebe.png
date: 2020-05-09 11:33:30
tags:
- Gridea
categories:
- 写BUG日常
- 教程
---
Gridea 是一款很好用的博客写作软件。它功能全面但可惜部署不支持国内的 Gitee 代码储存仓库。本篇文章将对 Gridea 进行源码上的修改使其支持 Gridea。
<!--more-->
{% raw %}<article class="message is-success"><div class="message-body">{% endraw %}
本篇文章会手把手进行教学，当然你也可以直接克隆已经做好的[仓库](https://gitee.com/Nofated/gridea)
{% raw %}</div></article>{% endraw %}

## 教程

### 克隆源代码

```
# Github 官方
git clone https:/github.com/getgridea/gridea.git

# FastGit 镜像
git clone https://hub.fastgit.org/getgridea/gridea.git

# Gitee 镜像
git clone https://gitee.com/mirrors/gridea.git

# Gitee
git clone https://gitee.com/fehey/gridea.git
```

以上四个仓库都是可以的。你可以自行选择。

```
# 进入 Gridea 的目录
cd gridea
```

## 修改源码

``` diff src/server/deploy.ts >folded
    this.platformAddress = ({  // 18行
      github: 'github.com',
      coding: 'e.coding.net',
+     gitee: 'gitee.com',
    } as any)[setting.platform || 'github']
  
    const preUrl = ({
      github: `${setting.username}:${setting.token}`,
      coding: `${setting.tokenUsername}:${setting.token}`,
+     gitee: `${setting.username}:${setting.token}`,
    } as any)[setting.platform || 'github']

    this.remoteUrl = `https://${preUrl}@${this.platformAddress}/${setting.username}/${setting.repository}.git`
```

``` diff src/server/events/deploy.ts>folded
    ipcMain.removeAllListeners('remote-detected') // 17行

    ipcMain.on('site-publish', async (event: IpcMainEvent, params: any) => {
+     console.log(platform)
      const client = ({
        'github': deploy,
        'coding': deploy,
+       'gitee': deploy,
        'sftp': sftp,
      } as any)[platform]


      const client = ({ // 38行
        'github': deploy,
        'coding': deploy,
+       'gitee': deploy,
        'sftp': sftp,
      } as any)[platform]
      
      const result = await client.remoteDetect()
      event.sender.send('remote-detected', result)
    })
```

``` diff src/views/setting/includes/BasicSetting.vue >folded
        <a-radio-group name="platform" v-model="form.platform"> // 5行
          <a-radio value="github">Github Pages</a-radio>
          <a-radio value="coding">Coding Pages</a-radio>
+         <a-radio value="gitee">Gitee Pages</a-radio>
          <a-radio value="sftp">SFTP</a-radio>
        </a-radio-group>
      </a-form-item>
          <a-input v-model="form.domain" placeholder="mydomain.com" style="width: calc(100% - 96px);" />
        </a-input-group>
      </a-form-item>
-     <template v-if="['github', 'coding'].includes(form.platform)">
+     <template v-if="['github', 'coding', 'gitee'].includes(form.platform)">
        <a-form-item :label="$t('repository')" :labelCol="formLayout.label" :wrapperCol="formLayout.wrapper" :colon="false">
          <a-input v-model="form.repository" />
        </a-form-item>
    export default class BasicSetting extends Vue {


      && form.branch // 134行
      && form.username
      && form.token
-   const pagesPlatfomValid = baseValid && (form.platform === 'github' || (form.platform === 'coding' && form.tokenUsername))
+   const pagesPlatfomValid = baseValid && (form.platform === 'gitee' || form.platform === 'github' || (form.platform === 'coding' && form.tokenUsername))
    const sftpPlatformValid = ['sftp'].includes(form.platform)
      && form.port
```

## 测试与编译

``` 需要安装 yarn 和 electron
yarn
# 测试
yarn electron:serve
# 生成
yarn electron:build
```