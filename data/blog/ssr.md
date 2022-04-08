---
title: React vite ts koa ssr
date: '2022-04-08'
tags: ['ssr', 'react', 'vite', 'ts', 'koa']
draft: false
summary: 使用 vite、ts、koa构建 react ssr服务
---

## 主要代码
```js
// app.ts
import Koa from 'koa'
import * as fs from 'fs'
import { resolve } from 'path'
import { createServer as createViteServer } from 'vite'

async function createServer() {
  const vite = await createViteServer({
    server: {
      open: '/index.html',
      middlewareMode: true,
    },
  })
  const app: Koa = new Koa()

  app.use(async (ctx, next) => {
    const url = ctx.originalUrl
    try {
      // 1. 应用 Vite HTML 转换。这将会注入 Vite HMR 客户端，
      //    同时也会从 Vite 插件应用 HTML 转换。
      //    例如：@vitejs/plugin-react 中的 global preambles
      const template = await vite.transformIndexHtml(
        url,
        fs.readFileSync(resolve(__dirname, '../index.html'), 'utf-8')
      )

      // 2. 加载服务器入口。vite.ssrLoadModule 将自动转换
      //    你的 ESM 源码使之可以在 Node.js 中运行！无需打包
      //    并提供类似 HMR 的根据情况随时失效。
      const { render } = await vite.ssrLoadModule('/src/entry-server.tsx')

      // 3. 渲染应用的 HTML。这假设 entry-server.js 导出的 `render`
      //    函数调用了适当的 SSR 框架 API。
      //    例如 ReactDOMServer.renderToString()
      const appHtml = render(url)

      // 4. 注入渲染后的应用程序 HTML 到模板中。
      const html = template.replace(`<!--ssr-outlet-->`, appHtml)

      // 5. 返回渲染后的 HTML。
      ctx.set({ 'Content-Type': 'text/html' })
      ctx.status = 200
      ctx.body = html
    } catch (e) {
      // 如果捕获到了一个错误，让 Vite 来修复该堆栈，这样它就可以映射回你的实际源码中。
      // @ts-ignore
      vite.ssrFixStacktrace(e)
    }
  })

  app.listen(3333, () => console.log('Server is running at 3333'))
}

createServer()

```

```js
// entry-server.tsx
import React from 'react'
import ReactDOMServer from 'react-dom/server'
import { StaticRouter } from 'react-router-dom/server'
import App from './App'

export function render(url: string) {
  return ReactDOMServer.renderToString(
    <React.StrictMode>
      <StaticRouter location={url}>
        <App />
      </StaticRouter>
    </React.StrictMode>
  )
}
```

```json
// package.json
{
  "name": "vite-project",
  "version": "0.0.0",
  "scripts": {
    "dev": "vite",
    "serve": "nodemon --watch src/**/* -e ts,tsx --exec ts-node ./server/app.ts",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "koa": "^2.13.4",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-router-dom": "^6.3.0"
  },
  "devDependencies": {
    "@types/koa": "^2.13.4",
    "@types/node": "^17.0.23",
    "@types/react": "^17.0.33",
    "@types/react-dom": "^17.0.10",
    "@vitejs/plugin-react-refresh": "^1.3.6",
    "nodemon": "^2.0.15",
    "ts-node": "^10.7.0",
    "tsconfig-paths": "^3.14.1",
    "typescript": "^4.6.3",
    "vite": "^2.7.2"
  }
}
```


```js
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": false,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": false,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": "./",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["./src"],
  // 这里是解决ts node 引用问题
  "ts-node": {
    "require": ["tsconfig-paths/register"],
    "compilerOptions": {
      "module": "commonJS"
    },
    "include": ["server/**/*.ts"]
  }
}
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/src/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="root"><!--ssr-outlet--></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```