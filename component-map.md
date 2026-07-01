# Component Map

当前没有真实前端项目，因此以下是生产实现建议，而不是对现有组件的准确引用。`index.html` 的 class 名不应直接当作生产组件名使用。

| UI element | Production component / style source | Reuse / new / modify | Related files | States | Notes |
| --- | --- | --- | --- | --- | --- |
| Page shell | New component needed: `OnlineListingMatchPage` | New | 未提供 | normal | 只承载 APP 列表、商品编辑、候选集三栏 |
| Status tabs | New or existing segmented tabs | Reuse if available | 未提供 | unmatched, matched | 未匹配增加数字角标 |
| Quick filter chips | New component needed: `FilterToggleChip` | New | 未提供 | on, off, disabled | 筛选标签是开关，不是 tab |
| Product list | New component needed: `ProductMatchList` | New | 未提供 | unmatched-card, matched-compare-card, selected, empty | 已匹配列表卡片内展示我的商品/外卖商品蓝黄对照 |
| Product detail form | Existing form/input/select if project has one | Modify | 未提供 | view, edit, dirty, reset, saving, validation-error | 商品条码字段可编辑；修改后显示重置按钮 |
| Barcode / UPC action | New component needed: `BarcodeFieldWithGuide` | New | 未提供 | valid, invalid, guide-open | 仅点击「商品条码」label 打开 UPC 引导 |
| Candidate list | New component needed: `TakeoutCandidateList` | New | 未提供 | high-confidence, low-confidence, duplicate, current | 当前关联置顶，疑似重复明确标识 |
| Result modal | Existing modal/dialog if available | Reuse | 未提供 | success, comfort, unlink-confirm | 不再使用旧匹配失败弹窗 |
| UPC guide drawer | Existing drawer/modal if available | Reuse / new | 未提供 | open, invalid, submitting | PC 可用右侧抽屉，避免遮住候选集 |
| Difference highlight | Existing text/status token if available | Reuse | 未提供 | mismatch | 已匹配页标红两边不一致资料 |
| Toast / inline feedback | Existing message system if available | Reuse | 未提供 | info, success, error | 错误尽量靠近字段或候选，而非只有 toast |
| Action buttons | Existing button system | Reuse | 未提供 | primary, secondary, danger, disabled, loading | 最后一条时隐藏「找不到，先看下一条」 |

## Icon Source

未发现项目图标库。生产实现应优先使用项目已有图标或 `lucide` 等统一图标源。Standalone HTML 使用内联 SVG 仅作为视觉参考。
