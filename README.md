# AI错题学伴

这是一个面向初中生的数学错题追问式 AI 学伴本地演示项目。

项目采用：

- `client/`：uni-app + Vue3，微信小程序前端
- `server/`：Node.js 本地后端，负责代理 Coze 或返回本地模拟结果
- 本地运行，不依赖 uniCloud，不需要部署到云服务器

## 功能

- 首页查看本地 AI 服务状态。
- 拍照或从相册选择错题图片。
- AI 识别题目和学生解题痕迹。
- AI 以多轮追问方式引导学生订正，支持“我不会”逐级提示。
- 学生完成至少三轮思考后生成结构化思维断点诊断。
- 自动生成错题订正卡、同断点变式题和下次复习时间。
- 订正卡保存到小程序本地历史记录。
- 展示个人错法画像和复习任务。

## 运行后端

```bash
cd server
npm install
npm run dev
```

后端默认运行在：

```text
http://127.0.0.1:3000
```

如果提示 `EADDRINUSE` 或 `address already in use :::3000`，说明 3000 端口已有旧后端在运行。可以先执行：

```bash
cd server
npm run stop
npm run dev
```

没有配置 Coze 时，后端会自动使用本地模拟模式，方便先录制演示或检查页面流程。

## 运行小程序

```bash
cd client
npm install
npm run dev:mp-weixin
```

然后用微信开发者工具导入：

```text
client/dist/dev/mp-weixin
```

也可以先构建：

```bash
cd client
npm run build:mp-weixin
```

再导入：

```text
client/dist/build/mp-weixin
```

本地调试时，请在微信开发者工具中勾选：

```text
不校验合法域名、web-view 业务域名、TLS 版本以及 HTTPS 证书
```

## 接入 Coze

复制配置文件：

```bash
cd server
copy .env.example .env
```

然后编辑 `server/.env`：

```text
PORT=3000
TUTOR_UNIT_NAME=一次函数
COZE_API_BASE_URL=https://api.coze.cn
COZE_API_TOKEN=你的 Coze API Token
COZE_BOT_ID=你的 Bot ID
COZE_WORKFLOW_ID=你的图片识别 Workflow ID
```

保存后重启后端：

```bash
npm run dev
```

首页状态显示 `Coze 已接入`，就说明小程序会通过本地后端调用 Coze。拍照分析优先使用 `COZE_WORKFLOW_ID`；如果只填写 `COZE_BOT_ID`，文字对话接口可用。

## 后端接口

已实现方案中的第一版接口：

```text
GET  /api/health
POST /api/photo-solve
POST /api/photo-solve-upload
POST /api/photo-analyze
POST /api/follow-up
POST /api/diagnosis
POST /api/correction-card
POST /api/variant-question
GET  /api/profile
```

保留兼容接口：

```text
POST /api/chat
POST /api/generate-card
```

## 目录说明

```text
client/src/pages/index/index.vue       首页
client/src/pages/tutor/tutor.vue       拍照分析与多轮追问
client/src/pages/card/card.vue         错题订正卡
client/src/pages/history/history.vue   历史记录
client/src/pages/profile/profile.vue   个人错法画像
client/src/pages/review/review.vue     复习任务
client/src/utils/api.js                前端请求封装
client/src/utils/storage.js            本地记录、画像和复习任务统计

server/src/app.js                      本地 HTTP 接口
server/src/analytics.js                诊断、订正卡、变式题和统计逻辑
server/src/cozeClient.js               Coze 聊天、文件上传和 Workflow 调用封装
server/src/prompt.js                   追问、订正卡和本地模拟逻辑
server/test/                           后端测试
```

## 测试

后端：

```bash
cd server
npm test
```

前端工具函数：

```bash
cd client
npm test
```

小程序构建：

```bash
cd client
npm run build:mp-weixin
```

当前测试覆盖：

- 无 Coze 配置时进入本地模拟模式。
- `/api/chat` 返回追问回复和订正卡草稿。
- `/api/photo-solve` 返回图片识别结果和订正卡草稿。
- `/api/follow-up` 至少三轮后才允许生成订正卡。
- `/api/diagnosis` 返回结构化思维断点。
- `/api/correction-card` 返回复习时间和变式题。
- `/api/profile` 返回个人错法画像统计结果。
- 前端本地登录信息生成逻辑。

## 登录说明

小程序启动后会先进入登录页。点击“微信授权注册/登录”会调用微信资料授权，并把头像、昵称和本次 `uni.login` 返回的 `code` 保存到本地缓存。第一次授权就视为本地注册，之后打开小程序会直接进入首页。

这个项目目前用于本地演示，不会上线，也没有真实用户数据库，所以不会用 `code` 去微信服务端换 `openid`。如果只是演示流程，可以点击“本地演示登录”直接进入首页。

真实上线时需要补充后端登录接口：小程序把 `code` 发送给后端，后端使用小程序 `appid` 和 `secret` 调用微信 `code2Session`，拿到 `openid` 后再建立自己的用户会话。
