---
title: 'React Native Client for NodeBB'
date: '2024/02/10'
lastmod: '2024/02/10'
tags: ['NodeBB', 'React Native', 'Open Source']
draft: false
summary: 'This is an open source project that I have been working on for a while. It is a React Native client for NodeBB. It is still in the early stages of development, but it is already functional and can be used to browse and interact with NodeBB forums. I would love to get some feedback and contributions from the community. You can find the project on GitHub at'
images:
  ['https://repository-images.githubusercontent.com/9603889/e174fda2-a71b-42e8-af15-e57d178ddcc6']
authors: ['default']
layout: PostLayout
---

## 背景

[Nodebb](https://nodebb.org/) 是一个开源的论坛系统，它是基于 Node.js 的，使用 MongoDB 或 Redis 作为数据库。它的界面是基于 Bootstrap 的，所以在移动端上的体验并不是很好。所以我决定开发一个 React Native 的客户端来提供更好的移动端体验。

## 项目演示

## 项目需求

NodeBB 并不是一个为了移动端而设计的论坛系统，如果你看过它的技术文档后，有些功能我们不得不自己进行定制：

1. 账户体系：NodeBB 有自己的帐户体系，使用邮箱和密码注册，但是在移动端使用第三方帐号登录（比如 Google，Apple）是更常见的做法，更便捷也更安全。这里我推荐使用 Google 的 React Native Firebase,它提供了一套完整的解决方案。
2. 接口鉴权：NodeBB 提供了一套 RESTful API，又分为 Read API 和 Write API, 调用接口需要 VerifyToken 参数，这个参数目前只能在后台管理界面手动创建，我们需要对代码进行修改，使得用户第三方帐号登录之后，自动获取 VerifyToken，用于后面的接口调用。

然后就是在客户端实现论坛系统的各种功能，初期我会先实现一些基本的功能，比如浏览帖子，发帖，回帖，点赞，举报，通知等等。

## 项目实现

### 1.帐户体系和接口鉴权问题

首先我们来看服务端有哪些需要改动。在 NodeBB 项目的`src/routes/write/index.js:54`新增一个接口，用于交换 NodeBB 系统的 verifyToken，代码如下：

```javascript
Write.reload = async (params) => {
  ...
  // @alin 重要功能，使用firebase.idToken 交换nodebb系统的verifyToken
  setupApiRoute(router, 'get', '/api/v3/exchangeVerifyToken', writeControllers.utilities.exchangeVerifyToken);
  ...
};
```

然后我们在`src/controllers/write/utilities.js`新增一个 exchangeVerifyToken 方法，FirebaseService 是我们新增的 service 类，里面包括很多 Firebase 相关的业务代码。代码如下：

```javascript
Utilities.exchangeVerifyToken = async (req, res) => {
  try {
    const idToken = req.get('idToken')
    const result = await FirebaseService.exchangeVerifyToken(idToken)
    helpers.formatApiResponse(200, res, result)
  } catch (error) {
    helpers.formatApiResponse(401, res, error.message)
  }
}
```

这里我们需要安装 firebase-admin

```bash
npm install firebase-admin
yarn add firebase-admin
```

然后我们需要在 Firebase 控制台创建一个项目，然后下载一个 json 文件，里面包含了一些重要的信息，比如 projectId, storageBucket 等等,保存到项目的合适位置。

代码的主要逻辑就是

1. 我们使用`firebaseAuth.verifyIdToken`方法来验证从客户端发送过来的 idToken，如果成功的话将得到一个 decodedToken，里面包含了一些重要的信息，比如 uid,email, displayName 等等。
2. 然后我们根据这些信息来创建或者查找用户，返回一个 NodeBB 生成的 uid
3. 然后我们再根据这个 uid 来创建或者查找一个 verifyToken，最后返回将 uid 和 verifyToken 返回给客户端。

到这里我们就完成了第一个功能，客户端使用第三方帐号登录成功之后，使用 idToken 去换取 NodeBB 系统的 verifyToken，用于后面的接口调用。

```javascript
'use strict'

const firebaseAdmin = require('firebase-admin')
const winston = require('winston')
const serviceAccount = require('./{YOUR_SERVICE_ACCOUNT}.json')
const User = require('../user')
const db = require('../database')
const apiUtils = require('../api/utils')
const utils = require('../utils')

const firebaseApp = firebaseAdmin.initializeApp({
  credential: firebaseAdmin.credential.cert(serviceAccount),
  projectId: '{YOUR_PROJECT_ID}',
  storageBucket: '{YOUR_STORAGE_BUCKET}',
})
const firebaseAuth = firebaseApp.auth()

const FirebaseService = module.exports

const findOrCreateUser = async (idToken, username, email) => {
  let uid = await User.getUidByEmail(email)
  if (!uid) {
    uid = await User.create({ username: username, email: email })
    await User.setUserField(uid, 'email', email)
    await User.email.confirmByUid(uid)
    // Save google-specific information to the user
    User.setUserField(uid, 'idToken', idToken)
    db.setObjectField('idToken:uid', idToken, uid)
  }
  return uid
}

const findOrCreateVerifyToken = async (uid) => {
  let verifyToken = await apiUtils.tokens.getTokenByUid(uid)
  if (!verifyToken) {
    verifyToken = await apiUtils.tokens.generate({ uid: uid, description: 'api access token' })
  }
  return verifyToken
}

FirebaseService.exchangeVerifyToken = async (idToken) => {
  if (!idToken) {
    throw new Error('idToken is required')
  }
  const decodedToken = await firebaseAuth.verifyIdToken(idToken)
  if (!decodedToken) {
    throw new Error('no decodedToken')
  }
  if (!decodedToken.email) {
    throw new Error('no email in decodedToken')
  }
  // split the email to get the username
  const username = decodedToken.email.split('@')[0]
  const uid = await findOrCreateUser(decodedToken.uid, username, decodedToken.email)
  const verifyToken = await findOrCreateVerifyToken(uid)
  return {
    uid: uid,
    verifyToken: verifyToken,
  }
}
```

接着我们来看客户端的实现。我们将使用[React Native Firebase](https://rnfirebase.io/)这个库来实现第三方帐号登录。首先我们需要安装这个库

```bash
# Using npm
npm install --save @react-native-firebase/app

# Using Yarn
yarn add @react-native-firebase/app
```

另外还有一些环境配置，具体可以参考[官方文档](https://rnfirebase.io/)。

我们需要为 axios 添加一个拦截器，用于在请求头添加 idToken 和 verifyToken，代码如下：

```javascript
import axios from 'axios'
import auth from '@react-native-firebase/auth'
import { MMKV } from 'react-native-mmkv'
const storage = new MMKV()

const axiosInstance = axios.create({
  baseURL: process.env.NODEBB_API_URL,
  timeout: 30000,
})

// 请求拦截器，添加 idToken和verifyToken到 headers 中
axiosInstance.interceptors.request.use(
  async function (config) {
    if (auth().currentUser != null) {
      const idToken = await auth().currentUser?.getIdToken()
      config.headers.idToken = idToken
    }
    const verifyToken = storage.getString('user.verifyToken')
    if (verifyToken) {
      config.headers.Authorization = `Bearer ${verifyToken}`
    }
    return config
  },
  function (error) {
    return Promise.reject(error)
  }
)
export default axiosInstance
```

我们新建一个 AuthContext.tsx 文件，用于管理用户的登录状态, 其中 Google 登录的逻辑如下：

```javascript
export function AuthProvider({ children }: { children: any }) {
  const googleSignIn = async () => {
    try {
      await GoogleSignin.hasPlayServices({
        showPlayServicesUpdateDialog: true,
      })
      //1. 获取用户的idToken
      const { idToken } = await GoogleSignin.signIn()
      const googleCredential = auth.GoogleAuthProvider.credential(idToken)
      await auth().signInWithCredential(googleCredential)
      //2. 刷新用户verifyToken和用户信息
      await refreshVerifyTokenAndUser()
    } catch (e) {
      console.error(e)
    }
  }
}
```

接着调用`/api/v3/exchangeVerifyToken`接口，将 idToken 发送到服务端，服务端返回 uid 和 verifyToken，然后我们将 verifyToken 保存到本地，用于后面的接口调用。至此我们解决了帐号体系问题和接口鉴权问题。

```javascript
export function AuthProvider({ children }: { children: any }) {
  const [verifyToken, setVerifyToken] = useMMKVString('user.verifyToken')
  const [user, setUser] = useMMKVObject < User > 'user'

  const refreshVerifyTokenAndUser = async () => {
    try {
      // 1.交换verifyToken
      const res = await UserAPI.exchangeVerifyToken()
      setVerifyToken(res.response.verifyToken)

      // 2.获取用户信息和保存deviceToken
      const [resUser, resDeviceToken] = await Promise.all([
        UserAPI.getUserByUid(res.response.uid),
        UserAPI.saveDeviceToken(deviceToken),
      ])
      setUser(resUser)
    } catch (e) {
      console.error(e)
    }
  }
}
```

### 2.论坛系统的功能实现

未完待续
