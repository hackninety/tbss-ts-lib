# tbss-ts-lib

> **维护位置已迁移**：后续源码、问题反馈和独立版本发布统一位于 [common-ts-lib/packages/tbss-ts-lib](https://github.com/hackninety/common-ts-lib/tree/main/packages/tbss-ts-lib)。包名与公开接口保留，本仓库保留历史，不再作为新版本安装来源。
>
> 当前安装：`npm install https://github.com/hackninety/common-ts-lib/releases/download/tbss-ts-lib-v0.1.0/tbss-ts-lib-0.1.0.tgz`
>
> 以下为迁移前的历史说明，其中旧仓库安装命令仅供追溯；新的开发和发布流程见 [迁移说明](https://github.com/hackninety/common-ts-lib/blob/main/MIGRATION.md)。


**铁板神数语料库**（一万二千条文十二集、十四考取数表十五张、注解精选 29 条、
方法文献三篇）—— 结构化 TypeScript 库，纯 TS、零运行时依赖、ESM、按集分包懒加载，
可直接被 React / Vite / Next / 纯 Node 消费。

**许可：CC BY-NC 4.0（禁止商用）**，详见 `LICENSE` —— 条文为清代公版古籍之社区
数字化转录，本库完整署名并锁定 commit 引用上游。**内容仅供文献研究与传统文化学习，
不构成任何命运预测依据。**

铁板神数（又作铁版神数），托名北宋邵雍的清代命理术数体系，以出生时间精确到刻
（15 分钟）的「考刻」推演著称，按数取条、依条断事；条文库共 12000 条，
按地支分子集至亥集十二集。

## 语料规模

| 分支 | 规模 | 保真度 | 底本 |
|---|---|---|---|
| 条文库 verses | 12000 条（十二集 × 1000，号 1001–13000 连续无缺；年龄注记 7089 条） | third-party 转录 | Nanphy/TiebanshenshuOS ← xaminxan/tiebanshenshu |
| 取数表 tables | 十四考 15 张 4799 数据行（月份/时辰/五音/日命/时运/考刻/卦象/先天命数/流年五表） | third-party 转录 | 同上（两上游逐字节同文） |
| 注解精选 annotated | 29 条 10 类（陈明点校本节录 + 今注/标签） | third-party 今注 | wangdwn/mingli-lab |
| 方法文献 docs | 概述/考刻/取数表说明 3 篇综述 | 本库编写 | 上游 README 与实现整理 |

底本引入锁定 commit（`getSources()`）；录入异常之修复（全库仅 4 处年龄注记分隔符缺失）
登记于 `getCorpusStats().textualIssues`。完整来源与取舍记录见 [docs/sources.md](docs/sources.md)。

## 安装与使用

```bash
npm install github:hackninety/tbss-ts-lib
```

```ts
// 根入口：注册表 + 常用条文面（轻量同步 + 懒加载）
import { getVolumes, getVerse, searchVerses, getCorpusStats, DISCLAIMER } from 'tbss-ts-lib';

getVolumes();               // 十二集（slug/地支/号段）
await getVerse(1001);       // { n:1001, volume:'子集', ages:[47], text:'一树残花，有枝复茂。' }
await searchVerses('姻缘'); // 繁简折叠全文检索（按集渐进加载）
```

```ts
// 条文库：卷册同步、载荷按集分十二包懒加载
import { getVolumeVerses, getVerses, getVersesByAge, loadAllVerses } from 'tbss-ts-lib/verses';

await getVolumeVerses('zi');       // 子集全 1000 条
await getVerses([2408, 8151]);     // 排盘流年批量取条
await getVersesByAge(47);          // 年龄倒排：47 岁应验条文 195 条
await loadAllVerses();             // 检索密集场景一次预热全库（约 0.8MB）
```

```ts
// 取数表：通用表格形态直接渲染 + 小型常量
import { getTables, getTable, EIGHT_QUARTERS, CORE_FORMULA, taixuanValue } from 'tbss-ts-lib/tables';

getTable('14-10');        // 先天命数表 { columns, rows }（多值单元格以 \n 分隔）
getTable('liunian-verse');// 语义 slug 双通道
EIGHT_QUARTERS;           // 八刻细分（刻别/分钟/天干/刻干数）
CORE_FORMULA;             // 终局条文数 = 本命基数 + 刻干数 × 48
taixuanValue('甲');       // 太玄数 9
```

```ts
// 注解精选与方法文献
import { getAnnotated, findAnnotated } from 'tbss-ts-lib/annotated';
import { getMethodDocs, getDocMarkdown } from 'tbss-ts-lib/docs';

getAnnotated();                    // 29 条（分类/标签/今注）
await getDocMarkdown('kaoke');     // 考刻方法综述 markdown

// RAG 出处组装
import { citeVerse, buildVerseContext } from 'tbss-ts-lib/prompt';
```

React 宿主接入（抽屉/检索/流年视图组件范式）见 [INTEGRATION.md](INTEGRATION.md)。

## 数据管线

```
docs/corpus/            规范底本（UTF-8；ingest 自上游锁定 commit 拉取）
  ├─ verses/tiaowen.csv       12000 条（集/条文号/年龄/断语）
  ├─ tables/14-*.csv          十四考十五张
  ├─ annotated/tiepan.json    注解精选
  └─ method/*.md              方法文献（手写综述）
        │  npm run gen
        ▼
src/data/*.json + src/chunks/*.ts     索引 + 分包载荷（生成文件勿手改）
        │  npm run build
        ▼
dist/                                  ESM + d.ts（相对导入已修扩展名，纯 Node 可直consume）
```

`npm run ingest` 重新拉取底本；`npm test` 校验统计锚点（12000/12/15/29/3）与检索行为。

## 声明

- 条文库无善本定本，坊间抄本文字互有出入；本库底本未经与清刊本逐字校勘
  （`FIDELITY_NOTE`），引用研究请核对原典。
- 铁板神数为历史术数文献。本库仅供文献研究、数字人文与传统文化学习之用，
  不构成任何命运预测依据，请勿据以做出人生决策（`DISCLAIMER`）。
