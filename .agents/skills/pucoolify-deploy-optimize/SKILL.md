---
name: pucoolify-deploy
description: |
  当用户提到"发布到 Pucoolify"、"部署到 Pucoolify"、"上传到 Pucoolify"、"发到 Pucoolify"、"帮我发布"、或在 Pucoolify 平台创建/部署应用时触发此技能。支持静态网站、Node.js、Python 等技术栈的部署。
  
  平台信息：
  - API 地址: http://api.demo.pucoolify.com
  - 支持技术栈: nodejs、python、static
  - 访问格式: http://{project_id}.demo.pucoolify.com
  
  默认账号信息：
  - 用户名: 18259083427
  - 密码: 123456
---

# Pucoolify 部署技能

将应用一键部署到 Pucoolify 轻量级 Kubernetes 平台。

## 工作流程

### 1. 认证登录
首先调用 GET /api/v1/ai/guide 了解平台信息，然后进行认证。
- 注册：`curl -X POST http://api.demo.pucoolify.com/api/v1/auth/register -H Content-Type: application/json -d "{\"username\":\"18259083427\",\"password\":\"123456\"}"`
- 登录：`curl -X POST http://api.demo.pucoolify.com/api/v1/auth/login -H Content-Type: application/json -d "{\"username\":\"18259083427\",\"password\":\"123456\"}"`
返回 access_token 用于后续请求，格式：`Authorization: Bearer token`

### 2. 项目管理
- **创建项目**：`curl -X POST http://api.demo.pucoolify.com/api/v1/projects -H Content-Type: application/json -H Authorization: Bearer token -d "{\"name\":\"项目名称\",\"description\":\"项目描述\",\"port\":80}"`
- **获取状态**：`curl -X GET http://api.demo.pucoolify.com/api/v1/projects/{project_id}/status -H Authorization: Bearer token`

### 3. 优化打包 (关键)
为了提高部署速度，必须在打包时排除冗余文件（如旧的代码包、Agent 日志等）：
```bash
zip -r code.zip . -x "code.zip" "*.git*" "node_modules/*" ".agents/*" "temp_restore/*" ".DS_Store"
```
*注意：必须包含 Dockerfile，最大文件大小 100MB。*

### 4. 上传与部署
`curl -X POST http://api.demo.pucoolify.com/api/v1/projects/{project_id}/upload -H Authorization: Bearer token -F file=@code.zip`
上传成功后自动触发构建。

### 5. 智能轮询策略
由于云端构建和 K8s 部署通常需要 **150-180 秒**，请采取以下策略：
1. 上传成功后，**首轮等待 60-90 秒**再进行第一次状态查询。
2. 若状态为 `building` 或 `deploying`，则每隔 30 秒查询一次。
3. 直到 `status` 变为 `running`。

## Dockerfile 模板
- **静态网站** (Port 80):
```dockerfile
FROM m.daocloud.io/docker.io/library/nginx:alpine
COPY . /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
- **Node.js** (Port 8080):
```dockerfile
FROM m.daocloud.io/docker.io/library/node:18-alpine
WORKDIR /app
RUN npm config set registry https://registry.npmmirror.com
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

## 常用接口
- 重新部署：`POST /api/v1/projects/{id}/redeploy`
- 停止项目：`POST /api/v1/projects/{id}/stop`

## 执行步骤
1. 使用默认账号登录获取 token。
2. 确认或创建项目 ID。
3. **执行优化打包命令**，排除冗余文件。
4. 上传代码并启动**智能轮询**。
5. 部署成功后返回 URL。
