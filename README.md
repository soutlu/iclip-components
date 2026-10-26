# iclip 组件库

成片编辑器与渲染服务用的 HyperFrames 组件库源目录。组装工具在公开仓库 `web/scripts/hyperframes-registry.mjs`；组装结果由部署目录的 `components/` 只读挂给 web 与 render 两个容器。增减组件只发布组件库，不发版。

## 目录

| 路径                 | 内容                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| `catalog.json`       | 组件目录：收哪些组件、标题、分类、可填项、封面；不写 `version`                                         |
| `registry.lock.json` | 组件与字体用到的 npm 包 tgz 的 sha256                                                                  |
| `patches/`           | 改写：`manifest.json` 记下每个文件替换的原文摘要，`null` 为新文件（自研组件都是新文件），文件放在同一相对路径 |
| `thumbnails/`        | 组件面板的封面                                                                                         |

## 组装

在公开仓库检出的 `web/` 下执行（先 `pnpm install`）：

```bash
pnpm hf:registry --library <本目录> --out <本机输出目录>
```

输出 `<本机输出目录>/catalog.json`（本目录的 `catalog.json` 加上 `version`）与 `<本机输出目录>/<版本号>/`。版本号是全部输入（本目录的四项、组装工具源码与 HyperFrames 版本）的摘要；同一版本号的目录已存在时不再组装，只重写 `catalog.json`。

改了组件目录、改写或升级了包之后，重写锁文件：

```bash
pnpm hf:registry --update-lock --library <本目录> --out <本机输出目录>
```

## 发布

服务器不访问本仓库，也不需要 git、node 或任何组装步骤。

1. 按上面的命令在本机组装。
2. 先上传版本目录：

   ```bash
   rsync -a <本机输出目录>/<版本号> <服务器>:<部署目录>/components/
   ```

3. 再上传 `catalog.json`：

   ```bash
   rsync -a <本机输出目录>/catalog.json <服务器>:<部署目录>/components/
   ```

   rsync 默认先写临时文件再改名，这一步是原子替换；不要加 `--inplace`。compose 挂载的是整个 `components/` 目录，改名替换后容器里立即看到新文件，不用重启。

旧版本目录留在服务器上，已打开的编辑器照常读取；不再需要时手动删除。
