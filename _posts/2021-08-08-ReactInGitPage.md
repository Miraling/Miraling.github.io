---
layout: post
title: '把现有的React项目部署到Github Pages'
date: 2021-08-08
author: 爱学习的lbl
color: rgb(89,255,172)
cover: 'https://www.programacion.com.py/wp-content/uploads/2016/11/react-logo-1024x576.png'
subtitle: '记录流程'
tags: 前端 React
music-id: 5189652
---

开始新的尝试，啦啦啦。嘻嘻，我设置的 BGM 每次都把我自己吓一跳 :)

## 本地项目

- 对项目使用

  ```
  npm install gh-pages --save-dev
  ```
  
  安装gh-pages。作用：通过 gh-pages 中间件可以把 build 文件下到文件推送到 github ，并且创建 gh-pages branch。
  
- 修改 package.json：
  
  第一步，增加 homepage ，把 github 远程项目仓库的" settings " ==> " Pages "中的网址填入。第二步，增加 npm scripts 命令，推送 gh-pages 。例如：
  
  ```javascript
  这是package.json
  {
  "name": "questionnaire-survey-platform",
    "version": "0.1.0",
    "private": true,
    "homepage": "https://miraling.github.io/questionnaire-survey-platform/",
    //添加了homepage
        
    "scripts": {
      "start": "react-scripts start",
      "build": "react-scripts build",
      "test": "react-scripts test",
      "eject": "react-scripts eject",
      "deploy": "gh-pages -d build"//添加这条
    },
  }
  ```
  
- 用 git 操作把代码同步到github仓库。

- 编译项目

  ```
  npm run build
  ```

  Github Pages不能识别react代码，要编译。

- 部署编译后的内容。
  
  ```
  npm run deploy
  ```
  
  


## GitHub 设置

- 在项目仓库的 **Settings** ==> **Pages**，选择 **Source** 来源为 **Branch : gh-pages /root** ，保存。



### 注意事项

主页的路由添加一个`exact`，写成:

```react
<Route exact path={'/'} component={Login}/>
```



后续想弄点后端api，maybe  [leancloud](https://leancloud.cn/) ?
