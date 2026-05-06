# 第 0 章：Hello World

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05-17/sections/00-hello-world/README.md`，基准提交：`c62d647`

先从基础 TypeScript 设置和一个 hello world 程序开始。

本指南使用 TypeScript 编写（没错，Python 版本很快会来）。

workshop 的每次文件修改之间都有很多 checkpoint，
所以即使你对 TypeScript 不是特别熟，
也应该能跟上并运行每个示例。

要运行本指南，你需要安装较新的 nodejs 和 npm。

你可以使用任何 nodejs 版本管理器，[homebrew](https://formulae.brew.sh/formula/node) 也可以。


    brew install node@20

你应该能看到 node 版本：

    node --version

复制初始 `package.json`：

    cp ./walkthrough/00-package.json package.json

<details>
<summary>显示文件</summary>

```json
// ./walkthrough/00-package.json
{
    "name": "my-agent",
    "version": "0.1.0",
    "private": true,
    "scripts": {
      "dev": "tsx src/index.ts",
      "build": "tsc"
    },
    "dependencies": {
      "tsx": "^4.15.0",
      "typescript": "^5.0.0"
    },
    "devDependencies": {
      "@types/node": "^20.0.0",
      "@typescript-eslint/eslint-plugin": "^6.0.0",
      "@typescript-eslint/parser": "^6.0.0",
      "eslint": "^8.0.0"
    }
  }
```

</details>

安装依赖：

    npm install

复制 `tsconfig.json`：

    cp ./walkthrough/00-tsconfig.json tsconfig.json

<details>
<summary>显示文件</summary>

```json
// ./walkthrough/00-tsconfig.json
{
    "compilerOptions": {
      "target": "ES2017",
      "lib": ["esnext"],
      "allowJs": true,
      "skipLibCheck": true,
      "strict": true,
      "noEmit": true,
      "esModuleInterop": true,
      "module": "esnext",
      "moduleResolution": "bundler",
      "resolveJsonModule": true,
      "isolatedModules": true,
      "jsx": "preserve",
      "incremental": true,
      "plugins": [],
      "paths": {
        "@/*": ["./*"]
      }
    },
    "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
    "exclude": ["node_modules", "walkthrough"]
  }
```

</details>

添加 `.gitignore`：

    cp ./walkthrough/00-.gitignore .gitignore

<details>
<summary>显示文件</summary>

```gitignore
// ./walkthrough/00-.gitignore
baml_client/
node_modules/
```

</details>

创建 `src` 文件夹。

    mkdir -p src

添加一个简单的 hello world `index.ts`：

    cp ./walkthrough/00-index.ts src/index.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/00-index.ts
async function hello(): Promise<void> {
    console.log('hello, world!')
}

async function main() {
    await hello()
}

main().catch(console.error)
```

</details>

运行它来验证：

    npx tsx src/index.ts

你应该会看到：

    hello, world!
