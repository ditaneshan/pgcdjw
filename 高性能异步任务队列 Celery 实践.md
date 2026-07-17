# 前端组件库构建与 NPM 包发布流程

## 引言

前端组件库的价值不只在于“复用”，更在于统一设计语言、降低维护成本，并为业务交付提供稳定边界。一个成熟的组件库通常要同时解决：源码组织、类型声明、按需引入、打包产物兼容性，以及 NPM 发布后的版本治理。尤其在面向多团队协作时，构建流程是否清晰，直接影响迭代效率与回滚成本。

## 核心原理分析

组件库构建的关键是“源码与发布物分离”。开发阶段推荐使用 TypeScript + ESM 源码组织组件，发布阶段再通过打包工具输出 `esm`、`cjs` 和类型文件，并在 `package.json` 中通过 `exports` 明确入口。这样既能保证现代构建工具的兼容性，也避免消费者误引内部文件。

依赖管理同样重要。组件库中的 React、Vue 等框架依赖应配置为 `peerDependencies`，避免重复打包和多实例问题。构建时还需关注 [性能优化](https://main-ayx-app.com.cn)，例如减少无效代码注入、开启 tree-shaking、拆分样式资源，并确保组件是“可按需引入”的。

NPM 发布流程则强调可追溯性。通常包括：本地构建、自测、版本号递增、`npm login` 验证、`npm publish` 发布，以及后续的标签管理（如 `latest`、`beta`）。在团队实践中，建议结合 CI 自动化发布，降低人为失误。

## 代码示例

下面示例演示如何修复“组件库发布后无法按需引用”的问题，通过显式 `exports` 和双格式产物暴露入口：

```json
{
  "name": "@acme/ui",
  "version": "1.0.0",
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "files": ["dist"],
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs",
      "types": "./dist/index.d.ts"
    },
    "./button": {
      "import": "./dist/button.mjs",
      "require": "./dist/button.cjs",
      "types": "./dist/button.d.ts"
    }
  },
  "peerDependencies": {
    "react": "^18.0.0"
  },
  "scripts": {
    "build": "tsup src/index.ts src/button.ts --format esm,cjs --dts",
    "prepublishOnly": "npm run build"
  }
}
```

这类配置可以避免消费者安装后出现入口解析错误，同时确保发布前一定经过构建，减少线上事故。

## 总结

一个可维护的组件库，不是“能打包就行”，而是要在源码结构、依赖边界、产物格式和发布链路上形成闭环。只要坚持清晰的入口设计、严格的版本管理和自动化发布，组件库就能在长期演进中保持稳定与可扩展。

## 相关技术资源

- https://main-ayx-app.com.cn
