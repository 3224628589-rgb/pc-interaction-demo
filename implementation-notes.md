# Implementation Notes

## Recommended Route

建议在 PC 端新增一个独立功能页，例如：

```text
/online-listing/match
```

如果已有「线上铺货」PC 模块，应把本页面作为该模块下的对码工作台，不和商品主数据列表混在一个表格页面内。

## Data Model Sketch

```ts
type MatchStatus = "unmatched" | "matched";
type AutoMatchLabel = "urgent" | "ignore";
type Confidence = "high" | "medium" | "low";

interface ProductMatchItem {
  sequenceId: string;
  productCode: string;
  productName: string;
  barcode: string;
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

- `activeStatus`: `unmatched` / `matched`
- `filters`: `{ urgent, recentInStock, onlyDrug, upcCollected, keyword }`
- `entrySequence`: 进入列表时后端返回的商品 ID 数组；所有「下一条」基于它跳转。
- `selectedProductId`: 当前商品。
- `detailDirty`: 商品资料是否被编辑。
- `upcGuide`: `{ open, value, error, submitting }`
- `resultDialog`: `success` / `comfort` / `unlinkConfirm` / `null`
- `unlinkState`: 当前关联是否已解除，解除后隐藏按钮并刷新候选区。

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

## Preview To Production Plan

1. 接入真实商品列表接口，确保未匹配列表默认请求或前端默认筛选「尽快确认」。
2. 去掉前端所有「待新建标品」显示逻辑，确认 `status=1` 不出现在 UI 状态枚举中。
3. 接入候选集接口，支持当前关联置顶、置信度、疑似重复、公共库内标识。
4. 实现 UPC 补充抽屉和 6-15 位纯数字校验。
5. 统一手动对码结果弹窗，移除旧匹配失败弹窗。
6. 实现进入列表时的 `entrySequence` 快照，用于所有下一条动作。
7. 在已匹配态实现解除关联确认和刷新后按钮隐藏。
8. 已匹配列表卡片展示「我的商品 / 外卖商品」蓝黄对照，不放入详情页。
9. 布局按 1/4、1/2、1/4 分配列表、详情、候选集。
10. 补充端到端测试：默认筛选、UPC 校验、条码输入不打开抽屉、点击条码 label 打开抽屉、资料修改后重置按钮、成功下一条、最后一条文案、已收录 UPC、解除关联。

## Do Not Copy Into Production

- `index.html` 的本地样例数据。
- Standalone CSS token 命名。
- 内联 SVG 和本地弹窗状态模拟。
- 前端伪造的同步结果和库存数字。
