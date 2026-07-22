# Implementation Notes

## Recommended Route

建议在 PC 端新增一个独立功能页，例如：

```text
/online-listing/match
```

如果已有「线上铺货」PC 模块，应把本页面作为该模块下的对码工作台，不和商品主数据列表混在一个表格页面内。

## Data Model Sketch

```ts
type MatchStatus = "unmatched" | "pending" | "synced";
type AutoMatchLabel = "priority" | "ignore";
type Confidence = "high" | "medium" | "low";

interface ProductMatchItem {
  sequenceId: string;
  productCode: string;
  productName: string;
  barcode: string;
  genericName: string;
  spec: string;
  approvalNo: string;
  manufacturer: string;
  category: "处方药" | "甲类非处方" | "乙类非处方" | "其他";
  status: MatchStatus;
  autoMatchLabel?: AutoMatchLabel;
  recentInStock?: boolean;
  onlyDrug?: boolean;
  upcCollected?: boolean;
  hasSaleableStock?: boolean;
  currentTakeoutProductId?: string;
  syncStatus?: boolean;
  hasRedDifference?: boolean;
}

interface TakeoutCandidate {
  id: string;
  name: string;
  upc: string;
  approvalNo: string;
  manufacturer: string;
  category: string;
  confidence: Confidence;
  duplicateSuspected?: boolean;
  inPublicLibrary?: boolean;
  isCurrent?: boolean;
}
```

## Frontend State

- `activeStatus`: `unmatched` / `pending` / `synced`
- `filters`: `{ priority, recentInStock, onlyDrug, upcCollected, hasRed, noRed, keyword }`
- `pendingSelectedIds`: 待同步勾选集合；有标红/无标红互斥。
- `entrySequence`: 进入列表时后端返回的商品 ID 数组；所有「下一条」基于它跳转。
- `selectedProductId`: 当前商品。
- `detailDirty`: 商品资料是否被编辑。
- `upcGuide`: `{ open, value, error, submitting }`
- `resultDialog`: `success` / `comfort` / `unlinkConfirm` / `null`
- `unlinkState`: 当前关联是否已解除，解除后隐藏按钮并刷新候选区。
- `disclaimer`: 门店+账号+版本维度的接受状态、首次 5 秒倒计时、正文版本。
- `tutorialStep`: 0-9；引导中顶部入口显示为 `退出引导`。

## Event Handlers

- `onSearch(keyword)`: 在当前状态 tab 和筛选开关下过滤商品。
- `onToggleFilter(key)`: 切换筛选开关；不是切换 tab。
- `onSelectProduct(id)`: 更新详情和候选；记录当前序列位置。
- `onSaveProductDetail(values)`: 保存商品资料，不自动确认候选。
- `onOpenUpcGuide()`: 仅点击 `商品条码` label 打开；点击条码输入框只编辑条码。
- `onResetProductDetail()`: 任意商品资料被修改后出现，点击恢复初始商品资料。
- `onSubmitUpc(upc)`: 校验 `^[0-9]{6,15}$`；失败显示「请输入合规upc」；成功关闭引导并触发手动对码。
- `onRematch()`: 根据当前资料重新匹配外卖商品。
- `onConfirmCandidate(candidateId)`: 弹成功确认；确认后按序列跳下一条，最后一条不跳。
- `onSkipToNext()`: 只在非最后一条展示，按序列进入下一条核对详情。
- `onUnlinkCurrent()`: 已匹配态弹确认；解除后刷新当前详情并隐藏按钮。
- `onSyncOne(id)` / `onSyncSelected(ids)`: 将是否同步改为是，并把商品从待同步移到已同步。
- `onAcceptDisclaimer()`: 首次倒计时结束后提交接受状态；再次查看立即可关闭。
- `onTutorialNext()`: 推进 10 步 PC 浮层并切换到对应状态。

## Preview To Production Plan

1. 列表接口支持未匹配/待同步/已同步，并返回各状态提示、是否同步和差异标红。
2. 未匹配 `优先确认` 默认关闭；待同步实现互斥筛选、勾选和单条/批量同步。
3. 候选接口使用专题新标签映射；候选资料来源由后端按静态库优先处理。
4. 手动匹配、建联和重关联按钮统一为“并同步”，成功后刷新三个 tab 数量和列表。
5. 接入门店+账号+版本维度免责声明；正文由后端按新/老门店返回。
6. 实现 10 步教程浮层，PC 上只高亮既有三栏区域。
7. 保留 UPC 抽屉、下一条序列、重置初始资料和解除关联状态迁移。
8. 补充端到端测试：免责倒计时、互斥筛选、单条/批量同步、搜索外卖资料、新标签、新弹窗、10 步教程、三状态流转。

## Do Not Copy Into Production

- `index.html` 的本地样例数据。
- Standalone CSS token 命名。
- 内联 SVG 和本地弹窗状态模拟。
- 前端伪造的同步结果和库存数字。
