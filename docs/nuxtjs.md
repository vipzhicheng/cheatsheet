Nuxt.js 速查表
===

[![NPM version](https://img.shields.io/npm/v/nuxt/latest.svg?style=flat)](https://www.npmjs.com/package/nuxt)
[![Downloads](https://img.shields.io/npm/dm/nuxt.svg?style=flat)](https://www.npmjs.com/package/nuxt)
[![Repo Dependents](https://badgen.net/github/dependents-repo/nuxt/nuxt)](https://github.com/nuxt/nuxt/network/dependents)
[![Github repo](https://badgen.net/badge/icon/Github?icon=github&label)](https://github.com/nuxt/nuxt)

这是一份快速参考速查表，包含 [Nuxt.js](https://nuxt.com/) 的核心 API 参考列表和一些示例。
<!--rehype:style=padding-top: 12px;-->

入门
----

### 创建项目

使用 `nuxi`（Nuxt CLI）创建新项目：

```shell
npx nuxi init <project-name>
# or
yarn dlx nuxi init <project-name>
# or
pnpm dlx nuxi init <project-name>
```

进入项目目录并安装依赖：

```shell
cd <project-name>
npm install
# or
yarn install
# or
pnpm install
```

运行 `npm run dev` 或 `yarn dev` 或 `pnpm dev` 以在 <http://localhost:3000> 上启动开发服务器。

### 添加首页

使用以下内容创建或填充 `pages/index.vue`：

```html
<template>
  <div>Welcome to Nuxt.js!</div>
</template>

<script setup>
// Nuxt 3 中，<script setup> 是推荐的写法
// 无需显式导出组件
</script>
```

Nuxt.js 也是围绕页面的概念构建的。 页面是 `pages` 目录中的 `.vue` 文件。 Nuxt 会自动根据文件结构生成路由。

### useFetch / useAsyncData (通用数据获取)
<!--rehype:wrap-class=row-span-2-->

Nuxt 3 推荐使用 `useFetch` 或 `useAsyncData` 进行数据获取，它们在服务器端和客户端都能工作，并处理 SSR 数据传递。

```html
<template>
  <div>
    <p v-if="pending">Loading...</p>
    <pre v-else-if="error">Error: {{ error.message }}</pre>
    <pre v-else>{{ data }}</pre>
  </div>
</template>

<script setup>
// 'useFetch' 是 'useAsyncData' 加上 fetch 的快捷方式
const { data, pending, error, refresh } = await useFetch('/api/data', {
  // 可以提供选项，如 pick, transform, server, lazy 等
  // pick: ['id', 'title'] // 只选择需要的字段
});

// 或者使用 useAsyncData + $fetch (Nuxt 内置的 fetch 封装)
/*
const { data, pending, error, refresh } = await useAsyncData(
  'myData', // 唯一的 key
  () => $fetch('/api/data')
)
*/

// 如果需要在 setup 外或非 setup 脚本中使用，移除 await
// const { data, pending, error } = useFetch('/api/data');
</script>
```

- `useFetch` 和 `useAsyncData` 在服务器端执行以进行 SSR。获取的数据会被序列化并传递给客户端。
- 在客户端导航时，它们会在客户端执行以获取新数据。
- `pending`: 请求是否进行中。
- `error`: 请求是否出错。
- `data`: 返回的数据 (ref)。
- `refresh`: 手动重新获取数据的函数。
- `lazy: true`: 数据在客户端导航后加载，显示加载状态，不阻塞导航。
- `server: false`: 只在客户端获取数据。

### 静态站点生成 (SSG)
<!--rehype:wrap-class=row-span-2-->

Nuxt 3 通过 Nitro 服务器引擎支持强大的 SSG 功能。

1. **数据获取**: 仍然使用 `useFetch` 或 `useAsyncData` 获取页面数据。
2. **生成**: 运行 `npx nuxi generate` 命令。
3. **预渲染**:
    - Nuxt 会自动爬取项目中的内部链接（`<NuxtLink>`）并预渲染这些页面。
    - 对于动态路由（如 `pages/posts/[id].vue`），如果 Nuxt 无法自动发现所有可能的路径，你需要在 `nuxt.config.ts` 中配置预渲染路由：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    prerender: {
      // crawlLinks: true, // 默认开启，爬取链接
      routes: ['/', '/about', '/posts/1', '/posts/2'] // 手动指定需要预渲染的路由
      // 也可以异步获取路由列表
      /*
      async routes() {
        const posts = await $fetch('/api/posts'); // 假设有获取所有 post ID 的 API
        return posts.map(post => `/posts/${post.id}`);
      }
      */
    }
  }
})
```

- SSG 适用于内容不经常变动、需要极致性能和 SEO 的场景。
- 生成的结果是纯静态文件（HTML, JS, CSS），可以部署在任何静态托管服务上。

### 增量静态再生 (ISR) / Stale-While-Revalidate (SWR)
<!--rehype:wrap-class=row-span-2-->

Nuxt 3 (Nitro) 允许在页面或 API 级别配置缓存策略，实现类似 ISR/SWR 的效果。

在 `nuxt.config.ts` 中使用 `routeRules` 配置：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // 对 /blog/** 下的所有页面应用 ISR，缓存 60 秒
    '/blog/**': { isr: 60 }, // 单位：秒

    // 对特定页面应用 SWR，缓存 1 小时，后台更新
    '/products/special': { swr: 3600 },

    // 对 API 路由应用缓存
    '/api/posts': { swr: 600 },

    // 禁用 SSR，仅 CSR
    '/admin/**': { ssr: false },

    // 重定向
    '/old-page': { redirect: '/new-page' },

    // 添加 headers
    '/assets/**': { headers: { 'cache-control': 's-maxage=31536000' } },
  }
})
```

- `isr: <seconds>`: 在指定时间内提供缓存，过期后下一个请求会触发后台重新生成。
- `swr: <seconds>`: 在指定时间内提供缓存，过期后下一个请求**立即**返回旧缓存，同时触发后台重新生成。
- 这需要在支持 Node.js 或 Serverless 环境的托管平台（如 Vercel, Netlify, Cloudflare Workers）上部署 `nuxi build` 的输出 (`.output` 目录)。

### 客户端数据获取 (onMounted)

如果数据仅在客户端需要（例如，用户特定数据且不需要 SEO），可以在 `onMounted` 钩子中使用原生 `fetch` 或 `$fetch`。

```html
<template>
  <div>
    <p v-if="loading">Loading profile...</p>
    <div v-else-if="profile">
      <h1>{{ profile.name }}</h1>
      <p>{{ profile.bio }}</p>
    </div>
    <p v-else>No profile data</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const profile = ref(null)
const loading = ref(true)

onMounted(async () => {
  try {
    // $fetch 是 Nuxt 提供的 fetch 封装，更好用
    profile.value = await $fetch('/api/profile-data')
  } catch (error) {
    console.error('Failed to load profile data:', error)
  } finally {
    loading.value = false
  }
})
</script>
```

*注意：* 这种方式获取的数据不会用于 SSR，可能影响首屏加载内容和 SEO。通常推荐使用 `useFetch` 或 `useAsyncData` 并设置 `server: false` 或 `lazy: true` 来实现类似效果，同时利用 Nuxt 的状态管理和错误处理。

### 静态文件服务

Nuxt 在项目根目录的 `public/` 文件夹下提供静态文件。代码可以从根 URL (`/`) 引用这些文件。

```html
<template>
  <!-- public/avatar.png -->
  <img src="/avatar.png" alt="User Avatar" width="64" height="64">

  <!-- 使用 NuxtImg (如果安装了 @nuxt/image) -->
  <NuxtImg src="/avatar.png" alt="User Avatar" width="64" height="64" />
</template>

<script setup>
// 如果使用 @nuxt/image
// import { NuxtImg } from '#components' // 通常 Nuxt 会自动导入
</script>
```

### 支持的浏览器和功能

Nuxt (基于 Vite) 默认目标是支持 [原生 ES 模块](https://caniuse.com/es6-module)、[原生 ESM 动态导入](https://caniuse.com/es6-module-dynamic-import) 和 [`import.meta`](https://caniuse.com/import-meta) 的现代浏览器。

- Chrome >=88
- Firefox >=78
- Safari >=14
- Edge >=88
<!--rehype:className=cols-3-->

可以通过 `nuxt.config.ts` 中的 `vite.build.target` 或 `.browserslistrc` 文件配置目标浏览器（Vite 会使用 `browserslist` 配置）。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    build: {
      target: 'es2015' // 或 'modules' (默认)
    }
  }
})
```

内置 CSS 支持
---

### 添加全局样式表
<!--rehype:wrap-class=row-span-2-->

在 `nuxt.config.ts` 的 `css` 数组中引入全局 CSS 文件。通常放在 `assets/css/` 目录下。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  css: [
    '~/assets/css/main.css', // 使用 ~ 代替 @ 指向 srcDir (通常是项目根目录)
  ],
})
```

例如，`assets/css/main.css`：

```css
body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  margin: 0 auto;
}
```

这些样式会被打包并全局应用。

### 从 node_modules 导入样式

同样在 `nuxt.config.ts` 的 `css` 数组中引入：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  css: [
    'bootstrap/dist/css/bootstrap.css', // 直接从 node_modules 引入
    '~/assets/css/main.css',
  ],
})
```

### 添加组件级 CSS (Scoped CSS & CSS Modules)
<!--rehype:wrap-class=row-span-2-->

Vue 组件天然支持 Scoped CSS 和 CSS Modules。

**Scoped CSS:** (最常用)

```html
<template>
  <button class="button error">Destroy</button>
</template>

<style scoped>
/* 样式只作用于当前组件 */
.button {
  padding: 8px 16px;
}
.error {
  color: white;
  background-color: red;
}
</style>
```

**CSS Modules:**

```html
<template>
  <!-- 使用 $style 对象绑定 class -->
  <button :class="$style.error">Destroy</button>
</template>

<style module>
/* 生成带 hash 的唯一类名, 如 .Button_error_hash */
.error {
  color: white;
  background-color: blue; /* 改为蓝色以区分 */
}
</style>
```

### Sass/SCSS/Less/Stylus 支持

Nuxt (通过 Vite) 内置了对 CSS 预处理器的支持。只需安装相应的预处理器即可。

```shell
# 安装 Sass
npm install --save-dev sass
# or yarn add --dev sass
# or pnpm add -D sass

# 安装 Less
npm install --save-dev less
# or yarn add --dev less
# or pnpm add -D less
```

安装后，可以直接在 `.vue` 文件或 `assets` 目录中使用 `.scss`, `.sass`, `.less`, `.styl` 文件。

```html
<style lang="scss" scoped>
$primary-color: #41B883; // Vue Green

.button {
  background-color: $primary-color;
  &:hover {
    opacity: 0.8;
  }
}
</style>
```

### 自定义预处理器选项
<!--rehype:wrap-class=col-span-2-->

在 `nuxt.config.ts` 中通过 `vite.css.preprocessorOptions` 配置。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    css: {
      preprocessorOptions: {
        scss: {
          // 自动导入全局变量/mixins
          additionalData: '@import "~/assets/scss/_variables.scss"; @import "~/assets/scss/_mixins.scss";',
        },
        less: {
          // Less 配置
          modifyVars: {
            'primary-color': '#1DA57A',
          },
          javascriptEnabled: true,
        }
      }
    }
  }
})
```

#### Sass 变量共享 (示例)

```scss
/* assets/scss/_variables.scss */
$primary-color: #3498db;
$secondary-color: #f1c40f;
```

然后在 `nuxt.config.ts` 中配置 `additionalData` 如上。之后就可以在任何 `.vue` 的 `<style lang="scss">` 中直接使用 `$primary-color`。

### CSS-in-JS (较少使用)

Vue 生态更倾向于使用 `<style scoped>` 或 CSS Modules。如果需要 CSS-in-JS，可以选择社区库如 `emotion` 或 `styled-components-vue`，并自行配置。

Layouts
---

### 基础示例 (默认布局)

创建 `layouts/default.vue` 文件。这个布局会自动应用到所有未使用其他布局的页面。

```html
// layouts/default.vue
<template>
  <div>
    <header>
      <nav>
        <NuxtLink to="/">Home</NuxtLink> |
        <NuxtLink to="/about">About</NuxtLink>
      </nav>
    </header>
    <main>
      <slot /> <!-- 页面内容会渲染在这里 -->
    </main>
    <footer>
      <p>© 2023 My Nuxt App</p>
    </footer>
  </div>
</template>
```

### 多个布局 (命名布局)

创建其他 `.vue` 文件在 `layouts/` 目录下，例如 `layouts/admin.vue`。

```html
// layouts/admin.vue
<template>
  <div class="admin-layout">
    <aside>Admin Sidebar</aside>
    <main>
      <slot />
    </main>
  </div>
</template>
```

### 在页面中指定布局

在页面的 `<script setup>` 中使用 `definePageMeta`：

```html
// pages/admin/dashboard.vue
<template>
  <div>Admin Dashboard Content</div>
</template>

<script setup>
definePageMeta({
  layout: 'admin' // 指定使用 admin.vue 布局
})
</script>
```

或者直接在页面模板中使用 `<NuxtLayout name="admin">`:

```html
// pages/admin/settings.vue
<template>
  <NuxtLayout name="admin">
    <div>Admin Settings Content</div>
    <!-- 也可以在这里添加布局特定的内容 -->
  </NuxtLayout>
</template>
```

*(注: 使用 `definePageMeta` 更常见和推荐)*

### 禁用布局

```html
// pages/login.vue - 不需要导航和页脚的页面
<template>
  <div>Login Form</div>
</template>

<script setup>
definePageMeta({
  layout: false // 禁用所有布局
})
</script>
```

### 数据请求 (在布局中)

可以在布局组件中获取共享数据。

```html
// layouts/default.vue
<template>
  <div>
    <header>
      <nav v-if="!pending && navigationData">
        <NuxtLink v-for="link in navigationData.links" :key="link.path" :to="link.path">
          {{ link.name }}
        </NuxtLink>
      </nav>
      <p v-else>Loading navigation...</p>
    </header>
    <main>
      <slot />
    </main>
    <footer>...</footer>
  </div>
</template>

<script setup>
// 获取在所有页面共享的导航数据
const { data: navigationData, pending } = await useFetch('/api/navigation', { lazy: true });
</script>
```

*注意：* 在布局中使用 `await` 会阻塞所有使用该布局的页面的渲染，直到数据获取完成。通常建议使用 `lazy: true` 或将数据获取移到 Pinia/状态管理中。

图片优化 (@nuxt/image)
---

需要先安装模块: `npm install -D @nuxt/image` 或 `yarn add -D @nuxt/image` 或 `pnpm add -D @nuxt/image`，然后在 `nuxt.config.ts` 中添加 `'@nuxt/image'` 到 `modules` 数组。

### 本地图片 (public/ 目录)

```html
<template>
  <h1>My Homepage</h1>
  <NuxtImg
    src="/me.png"
    alt="Picture of the author"
    width="500"
    height="500"
    format="webp"
    quality="80"
    placeholder="/placeholder.svg" <!-- 可选的占位符 -->
  />
  <p>Welcome!</p>
</template>
```

`@nuxt/image` 会自动优化图片，转换格式 (如 webp), 调整大小等。

### 远程图片

需要在 `nuxt.config.ts` 中配置允许的域名。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image'],
  image: {
    domains: ['images.unsplash.com', 'cdn.example.com'],
    // 可以为特定域名设置别名
    alias: {
      unsplash: 'https://images.unsplash.com'
    }
  }
})
```

```html
<template>
  <NuxtImg
    src="https://images.unsplash.com/photo-123"
    alt="Remote image"
    width="600"
    height="400"
    provider="ipx" <!-- Nuxt 内置的优化器，或配置其他如 Cloudinary, Vercel 等 -->
  />
  <!-- 使用别名 -->
  <NuxtImg src="/your-image-id" provider="unsplash" width="300" height="200" />
</template>
```

### Priority (预加载)

使用 `preload` 属性来提示浏览器优先加载关键图片 (如 LCP 元素)。

```html
<template>
  <NuxtImg
    src="/hero-banner.jpg"
    alt="Hero Banner"
    width="1200"
    height="600"
    preload
  />
</template>
```

### `<NuxtPicture>` 组件

用于艺术指导，根据不同屏幕尺寸提供不同图片源。

```html
<template>
  <NuxtPicture
    src="/image-large.png"
    :imgAttrs="{alt: 'My descriptive alt text'}"
    width="1000"
    height="500"
    :modifiers="{ resize: 'cover' }"
    :sizes="'xs:100vw sm:100vw md:100vw lg:1000px'"
    format="webp"
  />
</template>
```

优化字体
---

Nuxt 3 没有像 Next.js 13 那样内置深度集成的 `@next/font`。常用的方法有：

### 手动引入 (本地或 CDN)
<!--rehype:wrap-class=row-span-2-->

在全局 CSS 文件 (如 `assets/css/main.css`) 或布局组件的 `<style>` 中使用 `@font-face` 或 `@import`。

```css
/* assets/css/main.css */

/* 1. 使用 @font-face 引入本地字体文件 (放在 public/fonts/ 或 assets/fonts/) */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-v12-latin-regular.woff2') format('woff2'),
       url('/fonts/inter-v12-latin-regular.woff') format('woff');
  font-weight: 400;
  font-style: normal;
  font-display: swap; /* 推荐使用 swap */
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-v12-latin-700.woff2') format('woff2'),
       url('/fonts/inter-v12-latin-700.woff') format('woff');
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

/* 2. 或使用 @import 从 CDN 引入 (如 Google Fonts) */
/* @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap'); */

body {
  font-family: 'Inter', sans-serif;
}
```

确保在 `nuxt.config.ts` 中引入了 `main.css`。

### 使用 `@nuxtjs/google-fonts` 模块
<!--rehype:wrap-class=row-span-2-->

安装模块: `npm i -D @nuxtjs/google-fonts`，然后在 `nuxt.config.ts` 中配置。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/google-fonts'
  ],
  googleFonts: {
    families: {
      // 字体名称: [字重列表] 或 true (默认字重) 或 配置对象
      Roboto: [100, 300, 400, 500, 700, 900],
      Inter: {
        wght: [100, 300, 400, 700],
        ital: [100] // 斜体
      },
      'Josefin+Sans': true, // 加号替换空格
    },
    display: 'swap', // 默认值 'swap'
    prefetch: true, // 预获取字体链接
    preconnect: true, // 预连接 Google Fonts
    // download: true, // ！！！下载字体到本地并自托管 (推荐！)
    // useStylesheet: true // 使用 <link rel="stylesheet"> 代替 @import
    // base64: true // 如果 download 为 true, 可以内联 base64 字体 (不推荐，包体积大)
  }
})
```

该模块会自动处理字体的加载和优化，推荐开启 `download: true` 以获得最佳性能和隐私。

### 使用 Tailwind CSS

如果使用 `@nuxtjs/tailwindcss` 模块，可以在 `tailwind.config.js` 中配置字体。

```javascript
// tailwind.config.js
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  theme: {
    extend: {
      fontFamily: {
        // 假设已通过 @font-face 或 @nuxtjs/google-fonts 引入了 'Inter'
        sans: ['Inter', ...defaultTheme.fontFamily.sans],
      },
    },
  },
  // ... other config
}
```

优化 Scripts (`useHead`)
---

Nuxt 3 使用 `useHead` 组合式函数来管理 `<head>` 标签，包括 `<script>`。

### 页面特定脚本

在页面组件的 `<script setup>` 中使用 `useHead`。

```html
<template>
  <div>Dashboard Page</div>
</template>

<script setup>
useHead({
  script: [
    {
      src: 'https://example.com/dashboard-analytics.js',
      // defer: true, // 或 async: true
      // body: true // 将脚本放在 <body> 结束前，而不是 <head>
    }
  ]
})
</script>
```

### 应用全局脚本
<!--rehype:wrap-class=row-span-2-->

在 `nuxt.config.ts` 的 `app.head` 中配置全局脚本。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      script: [
        {
          src: 'https://example.com/global-tracker.js',
          async: true,
          // body: true, // 推荐放 body 底部
        }
      ]
      // 也可以添加 link, meta, title 等
    }
  }
})
```

或者在 `app.vue` 或 默认布局 `layouts/default.vue` 中使用 `useHead`。

### 内联脚本
<!--rehype:wrap-class=col-span-2-->

可以使用 `innerHTML` 或 `children` 属性添加内联脚本。

```typescript
// nuxt.config.ts (全局示例)
export default defineNuxtConfig({
  app: {
    head: {
      script: [
        {
          // children 属性更安全，Nuxt 会处理转义
          children: `console.log('Inline script executed');`,
          // id: 'my-inline-script',
          // type: 'text/javascript'
        },
        /*
        {
          // 或者使用 innerHTML (需要注意 XSS 风险)
          innerHTML: `document.getElementById('banner').style.display = 'block';`,
          // body: true
        }
        */
      ]
    }
  }
})
```

在组件中使用 `useHead`:

```html
<script setup>
useHead({
  script: [
    { children: `alert('Hello from inline script!');` }
  ]
})
</script>
```

### Script 事件 (onload, onerror)

`useHead` 目前不直接支持 `onload` 或 `onerror` 回调。如果需要，可以通过创建一个包装函数或使用第三方库（如 `@vueuse/head` 提供的更高级功能，但 Nuxt 的 `useHead` 通常足够）。对于简单场景，可以在 `onMounted` 中检查脚本是否加载（例如检查脚本定义的全局变量）。

```html
<script setup>
import { onMounted } from 'vue'

useHead({
  script: [
    { src: 'https://example.com/library.js', id: 'lib-script' }
  ]
})

onMounted(() => {
  const script = document.getElementById('lib-script');
  if (script) {
    script.onload = () => {
      console.log('Library script loaded!');
      // 可以在这里调用库的函数
      // if (window.myLibrary) { window.myLibrary.init(); }
    };
    script.onerror = () => {
      console.error('Failed to load library script.');
    };
  }

  // 或者轮询检查脚本是否定义了某个全局变量
  /*
  const interval = setInterval(() => {
    if (window.myLibrary) {
      console.log('Library ready!');
      window.myLibrary.init();
      clearInterval(interval);
    }
  }, 100);
  */
})
</script>
```

ESLint
---

### 集成 ESLint

Nuxt 3 推荐使用 `@nuxtjs/eslint-module`。

```bash
npx nuxi module add eslint --install
# 或者手动安装
# npm install --save-dev @nuxtjs/eslint-module eslint @nuxtjs/eslint-config-typescript
# yarn add --dev @nuxtjs/eslint-module eslint @nuxtjs/eslint-config-typescript
# pnpm add -D @nuxtjs/eslint-module eslint @nuxtjs/eslint-config-typescript
```

然后在 `nuxt.config.ts` 中添加模块：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/eslint-module'
  ],
  // 可选配置
  eslint: {
    lintOnStart: false, // 不在开发服务器启动时检查 (推荐 false 以加快启动)
    // cache: false,
  }
})
```

在 `package.json` 中添加 lint 脚本：

```json
"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix"
}
```

创建一个 `.eslintrc.cjs` (或 `.js`, `.json`) 配置文件：

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  extends: [
    'plugin:vue/vue3-essential', // Vue 3 基础规则
    '@nuxtjs/eslint-config-typescript' // Nuxt + TypeScript 推荐规则
    // 'prettier' // 如果使用 Prettier，确保这是最后一个 (见下文)
  ],
  parserOptions: {
    ecmaVersion: 'latest'
  },
  rules: {
    // 在这里覆盖或添加规则
    // "vue/multi-word-component-names": "off",
  }
}
```

运行 `npm run lint` 或 `yarn lint` 或 `pnpm lint`。

### 自定义配置

直接修改 `.eslintrc.cjs` 文件即可。

### 对自定义目录和文件进行检查

ESLint CLI 支持指定目录或文件：

```bash
# 只检查 components 和 composables 目录
eslint components composables

# 检查特定文件
eslint pages/index.vue utils/helpers.ts
```

可以在 `package.json` 中定义更具体的脚本。

### 禁用规则
<!--rehype:wrap-class=row-span-2-->

在 `.eslintrc.cjs` 的 `rules` 部分添加规则并设置为 `"off"` 或 `0`。

```javascript
// .eslintrc.cjs
module.exports = {
  // ...
  rules: {
    'vue/no-multiple-template-root': 'off', // 禁用 Vue 3 不再需要的规则
    '@typescript-eslint/no-unused-vars': ['warn', { 'argsIgnorePattern': '^_' }], // 未使用的变量警告（忽略下划线开头的）
  }
}
```

也可以使用行内注释禁用：

```javascript
// eslint-disable-next-line no-console
console.log('临时允许 console.log');
```

### Prettier 集成

安装 Prettier 和 ESLint 的集成插件：

```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
# yarn add --dev prettier eslint-config-prettier eslint-plugin-prettier
# pnpm add -D prettier eslint-config-prettier eslint-plugin-prettier
```

在 `.eslintrc.cjs` 的 `extends` 数组**末尾**添加 `'prettier'` (禁用与 Prettier 冲突的规则) 或 `'plugin:prettier/recommended'` (应用 Prettier 规则并报告差异为 ESLint 错误)。

```javascript
// .eslintrc.cjs (推荐方式)
module.exports = {
  root: true,
  extends: [
    'plugin:vue/vue3-essential',
    '@nuxtjs/eslint-config-typescript',
    'plugin:prettier/recommended' // 集成 Prettier
  ],
  // ...
}
```

创建 `.prettierrc.js` (或 `.json`, `.yaml`) 配置文件：

```javascript
// .prettierrc.js
module.exports = {
  semi: false, // 不加分号
  singleQuote: true, // 使用单引号
  trailingComma: 'es5', // 尾随逗号
  printWidth: 100, // 行宽
  // ... 其他 Prettier 选项
};
```

### lint-staged
<!--rehype:wrap-class=col-span-2-->

结合 Husky 和 lint-staged 可以在 Git 提交前自动检查和修复暂存文件。

安装：

```bash
npm install --save-dev husky lint-staged
# yarn add --dev husky lint-staged
# pnpm add -D husky lint-staged
npx husky install
npm set-script prepare "husky install" # 或 yarn husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

在 `package.json` 或 `.lintstagedrc.js` 中配置 `lint-staged`：

```json
// package.json
{
  "lint-staged": {
    "*.{js,ts,vue}": "eslint --fix",
    "*.{css,scss,vue}": "prettier --write"
  }
}
```

```javascript
// .lintstagedrc.js
module.exports = {
  '*.{js,ts,vue}': 'eslint --fix',
  '*.{css,scss,vue,json,md}': 'prettier --write',
}
```

TypeScript
---

Nuxt 3 是 TypeScript 优先的框架。

### 创建项目 (自带 TS)

`npx nuxi init <project-name>` 默认创建的就是 TypeScript 项目，包含 `tsconfig.json`。

### 类型检查

Nuxt 3 会自动生成类型 (`.nuxt/nuxt.d.ts`)，提供自动导入、配置、API 的类型提示。

运行类型检查：

```bash
npx nuxi typecheck
# 或者
npm run typecheck # (如果 package.json 中定义了脚本)
```

### 数据获取类型
<!--rehype:wrap-class=col-span-2 row-span-2-->

`useFetch` 和 `useAsyncData` 支持泛型来指定返回数据的类型。

```html
<script setup lang="ts">
interface Post {
  id: number;
  title: string;
  body: string;
}

// useFetch 的类型会自动从 API 响应推断 (如果 API 返回类型良好)
// 但显式指定更安全
const { data: post, pending, error } = await useFetch<Post>('/api/posts/1')

if (post.value) {
  console.log(post.value.title) // 类型安全
}

// useAsyncData 需要显式指定类型
const { data: posts } = await useAsyncData<Post[]>('postsList', () => $fetch('/api/posts'))

if (posts.value) {
  posts.value.forEach(p => console.log(p.id)); // 类型安全
}

// pick 选项也会影响类型
const { data: postTitle } = await useFetch('/api/posts/1', {
  pick: ['title'] // data 的类型会变成 Pick<Post, 'title'>
})
if (postTitle.value) {
  console.log(postTitle.value.title);
  // console.log(postTitle.value.id); // TS 报错，因为 id 没被 pick
}

// transform 选项改变类型
const { data: postLength } = await useFetch<Post>('/api/posts/1', {
  transform: (post): number => {
    return post.body.length;
  } // data 类型现在是 number | null
})
</script>
```

### API 路由 (Server Routes)
<!--rehype:wrap-class=row-span-2-->

API 路由位于 `server/api/` 目录下，使用 `defineEventHandler`。类型通常会自动推断。

```typescript
// server/api/hello.ts
import type { IncomingMessage, ServerResponse } from 'http'

// defineEventHandler 会处理请求和响应对象，简化类型
export default defineEventHandler(event => {
  // event 对象包含 request, response 等信息
  // event.node.req (原生 Node 请求对象)
  // event.node.res (原生 Node 响应对象)
  // event.context (请求上下文，可以添加中间件信息)

  // 读取查询参数
  const query = getQuery(event) // { name: 'World' } for /api/hello?name=World
  const name = query.name || 'World'

  // 读取请求体 (需要异步)
  // const body = await readBody(event)

  // 设置 header
  // event.node.res.setHeader('Content-Type', 'application/json')

  // 返回值会被自动序列化 (通常是 JSON)
  return {
    message: `Hello, ${name}!`
  }
})

// 也可以定义更具体的返回类型
interface HelloResponse {
  message: string;
}
export default defineEventHandler<HelloResponse>(event => {
   // ...
   return { message: 'Hello' }
})
```

### 自定义 `app.vue`

如果需要自定义 `app.vue` (不常见)，它就是一个标准的 Vue 3 TypeScript 组件。

```html
// app.vue
<template>
  <NuxtLayout>
    <NuxtPage /> <!-- 必须包含 NuxtPage -->
  </NuxtLayout>
</template>

<script setup lang="ts">
// 可以在这里添加全局逻辑或状态初始化
import { useHead } from '#imports' // Nuxt 自动导入

useHead({
  titleTemplate: (titleChunk) => {
    return titleChunk ? `${titleChunk} - My Nuxt App` : 'My Nuxt App';
  }
})
</script>
```

### 类型化 `nuxt.config.ts`

`defineNuxtConfig` 提供了完整的类型提示。

```typescript
// nuxt.config.ts
import { defineNuxtConfig } from 'nuxt/config'

export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: ['@nuxtjs/tailwindcss'],
  css: ['~/assets/css/main.css'],
  runtimeConfig: { // 运行时配置 (服务端可用)
    apiKey: 'default-secret-key', // process.env.NUXT_API_KEY 会覆盖
    public: { // 公开配置 (客户端和服务端都可用)
      baseUrl: '/api', // process.env.NUXT_PUBLIC_BASE_URL 会覆盖
    }
  },
  // ... 其他选项都有类型提示
})
```

### 忽略 TypeScript 错误

可以在 `nuxt.config.ts` 中配置，但不推荐用于生产构建。

```typescript {3-5}
// nuxt.config.ts
export default defineNuxtConfig({
  typescript: {
    // 在构建时忽略 TS 错误
    // typeCheck: false, // (旧版本或特定场景)
    shim: false // 是否生成 shim 文件
    // 推荐: 使用 `npx nuxi typecheck` 单独检查类型
  },
})
```

环境变量
---

### 加载环境变量

Nuxt 3 默认从项目根目录的 `.env` 文件加载环境变量。

```ini
# .env
DB_HOST=localhost
DB_USER=myuser
DB_PASS=secretpassword

# 暴露给客户端的变量需要 NUXT_PUBLIC_ 前缀
NUXT_PUBLIC_GOOGLE_ANALYTICS_ID=UA-1234567-8
NUXT_PUBLIC_API_BASE_URL=/api
```

### 使用环境变量 (Runtime Config)

推荐使用 **Runtime Config** 来访问环境变量，它提供了类型安全和更好的管理。

在 `nuxt.config.ts` 中定义 `runtimeConfig`:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // 只在服务端可用的私有键 (默认从 .env 读取)
    dbUser: process.env.DB_USER, // 也可以直接赋值
    dbPassword: process.env.DB_PASS,

    // public 中的键在客户端和服务端都可用
    public: {
      gaId: process.env.NUXT_PUBLIC_GOOGLE_ANALYTICS_ID,
      apiBaseUrl: '/api' // 默认值
    }
  }
})
```

在应用中使用 `useRuntimeConfig()`：

```html
<script setup lang="ts">
const config = useRuntimeConfig()

// 在 <script setup> 或 setup() 中 (服务端和客户端)
console.log('API Base URL:', config.public.apiBaseUrl)
console.log('GA ID:', config.public.gaId)

// 只能在服务端访问私有键 (例如 API 路由、服务器插件)
// console.log('DB User:', config.dbUser) // 在客户端会是 undefined

// 如果在服务端代码中 (如 server/api/ or server/plugins/)
// 可以直接访问 config.dbUser
if (process.server) {
  console.log('DB User (server-side):', config.dbUser)
}

// 在 API 路由中使用
// server/api/my-endpoint.ts
export default defineEventHandler(() => {
  const config = useRuntimeConfig()
  console.log('Accessing DB User in API:', config.dbUser)
  return { status: 'ok' }
})
</script>
```

### 变量扩展和转义

Vite (Nuxt 3 底层) 支持 `.env` 文件中的变量扩展，但行为可能与 Next.js 的 `dotenv-expand` 略有不同。通常建议避免复杂的变量扩展。

如果需要美元符号 `$`，通常不需要转义，但具体取决于上下文。

### 优先级

环境变量加载优先级：`.env.production` > `.env.development` > `.env` > 系统环境变量。 `process.env` 中的同名变量（如 `NUXT_PUBLIC_API_BASE_URL`）会覆盖 `runtimeConfig` 中的默认值。

路由
---

Nuxt 根据 `pages/` 目录结构自动生成 Vue Router 配置。

### 基础路由
<!--rehype:wrap-class=row-span-2-->

文件结构决定路由路径。

:-- | --
:-- | --
`pages/index.vue` | <pur>`/`</pur>
`pages/about.vue` | <pur>`/about`</pur>
`pages/blog/index.vue` | <pur>`/blog`</pur>
`pages/blog/first-post.vue` | <pur>`/blog/first-post`</pur>
`pages/users/index.vue` | <pur>`/users`</pur>
`pages/users/profile.vue` | <pur>`/users/profile`</pur>
<!--rehype:className=style-list-->

### 动态路由
<!--rehype:wrap-class=row-span-2-->

使用方括号 `[]` 定义动态参数。

:-- | --
:-- | --
`pages/users/[id].vue` | <pur>`/users/:id`</pur> (<yel>`/users/123`</yel>)
`pages/posts/[slug].vue` | <pur>`/posts/:slug`</pur> (<yel>`/posts/my-first-post`</yel>)
`pages/products/[category]/[productId].vue` | <pur>`/products/:category/:productId`</pur> (<yel>`/products/electronics/456`</yel>)
`pages/[username]/settings.vue` | <pur> `/:username/settings` </pur> (<yel>`/john_doe/settings`</yel>)
<!--rehype:className=style-list-->

在组件中访问路由参数：

```html
// pages/users/[id].vue
<template>
  <div>User ID: {{ $route.params.id }}</div>
</template>

<script setup>
import { useRoute } from 'vue-router' // 或 Nuxt 自动导入的 useRoute

const route = useRoute()
const userId = computed(() => route.params.id) // 使用 computed 获取响应式参数

console.log('User ID from setup:', route.params.id);
</script>
```

### 捕获所有路由 (Catch-all)
<!--rehype:wrap-class=row-span-2-->

使用带有三个点的方括号 `[...]`。

:-- | --
:-- | --
`pages/articles/[...slug].vue` | <pur>`/articles/*`</pur> (<yel>`/articles/a/b/c`</yel>)

参数值是一个数组。

```html
// pages/articles/[...slug].vue
<template>
  <div>Article Slug Parts: {{ slugParts }}</div>
</template>

<script setup>
const route = useRoute()
// route.params.slug 会是 ['a', 'b', 'c'] 对于 /articles/a/b/c
const slugParts = computed(() => route.params.slug || [])
</script>
```

### 可选捕获所有路由 (Optional Catch-all)

使用双方括号 `[[...]]`。这会匹配根路径以及之后的所有路径。

:-- | --
:-- | --
`pages/optional/[[...slug]].vue` | <pur>`/optional/*`</pur> (匹配 <yel>`/optional`</yel>, <yel>`/optional/a`</yel>, <yel>`/optional/a/b`</yel>)

- 访问 `/optional` 时, `route.params.slug` 是 `undefined`。
- 访问 `/optional/a` 时, `route.params.slug` 是 `['a']`。
- 访问 `/optional/a/b` 时, `route.params.slug` 是 `['a', 'b']`。

### 页面之间的链接 (`<NuxtLink>`)
<!--rehype:wrap-class=row-span-2-->

使用 `<NuxtLink>` 组件进行内部导航。

```html
<template>
  <nav>
    <ul>
      <li>
        <NuxtLink to="/">首页</NuxtLink>
      </li>
      <li>
        <NuxtLink to="/about">关于我们</NuxtLink>
      </li>
      <li>
        <!-- 链接到动态路由 -->
        <NuxtLink :to="`/users/${userId}`">我的主页</NuxtLink>
      </li>
      <li>
        <NuxtLink :to="{ name: 'posts-slug', params: { slug: postSlug } }">
          博文 (命名路由)
        </NuxtLink>
      </li>
      <li>
        <NuxtLink to="/external-page" external>外部链接</NuxtLink>
      </li>
    </ul>
  </nav>
</template>

<script setup>
const userId = ref(123)
const postSlug = ref('hello-nuxt')

// 可以通过 `npx nuxi routes` 查看生成的路由名称
// 例如 pages/posts/[slug].vue 的名称可能是 'posts-slug'
</script>

<style scoped>
/* NuxtLink 活动链接的默认 class */
.router-link-active {
  font-weight: bold;
}
/* NuxtLink 精确匹配活动链接的默认 class */
.router-link-exact-active {
  color: #41B883; /* Vue Green */
}
</style>
```

### 编程式导航 (`useRouter`)

使用 `useRouter` 获取 router 实例。

```html
<template>
  <button @click="goToAbout">跳转到关于页</button>
  <button @click="goToPost('my-latest-post')">查看最新帖子</button>
</template>

<script setup>
import { useRouter } from 'vue-router' // 或 Nuxt 自动导入

const router = useRouter()

function goToAbout() {
  router.push('/about')
}

function goToPost(slug) {
  // 使用路径
  // router.push(`/posts/${slug}`)

  // 或使用命名路由和参数 (推荐)
  router.push({ name: 'posts-slug', params: { slug } })
}
</script>
```

### 路由元信息 (`definePageMeta`)

在页面组件中使用 `definePageMeta` 添加路由元信息，可用于中间件、布局等。

```html
// pages/admin/index.vue
<script setup lang="ts">
definePageMeta({
  layout: 'admin', // 指定布局
  middleware: ['auth', 'admin-only'], // 应用中间件 (文件位于 middleware/ 目录下)
  // alias: ['/dashboard'], // 路由别名
  // keepalive: true, // 缓存页面状态
  // key: route => route.fullPath, // 自定义 key 用于 <NuxtPage :page-key="...">
  // transition: { name: 'fade', mode: 'out-in' } // 页面过渡效果
})
</script>
```

### 路由中间件

创建 `middleware/auth.ts` (或其他名称)：

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const isLoggedIn = false // 实际应用中应检查用户登录状态

  if (!isLoggedIn && to.path !== '/login') {
    // console.log(`Redirecting from ${from.path} to /login (accessed ${to.path})`)
    return navigateTo('/login') // Nuxt 提供的重定向辅助函数
  }
  // 如果已登录或访问的是登录页，则继续导航
})
```

然后在 `definePageMeta` 或 `nuxt.config.ts` (全局中间件) 中应用。

另见
---

- [Nuxt.js 官方文档](https://nuxt.com/docs)
- [Nuxt Community Modules](https://nuxt.com/modules)
- [Nitro Server Engine 文档](https://nitro.unjs.io/)
- [Vue 3 文档](https://vuejs.org/guide/introduction.html)
- [Vite 文档](https://vitejs.dev/)
