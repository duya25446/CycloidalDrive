# 摆线减速器脚本升级 - 实施计划

## 1. 目标 / 非目标
**目标**
- 在零件设计文档中不再新建组件，直接在 `rootComponent` 上创建草图，避免 `addNewComponent` 失败。
- 保持现有几何绘制与参数逻辑不变。

**非目标**
- 不修改数学算法与参数模型。
- 不重构 UI 或事件处理逻辑。

## 2. 关键资产保留 (Mandatory Technical Assets)
无。

## 3. 现状摘要
- `DrawCycloReducer.__init__` 中直接调用 `occurrences.addNewComponent`，在单组件设计下报错。
- 该逻辑是本次唯一切入点，后续草图绘制基于 `compReducer`。

## 4. 设计方案 (Design & Strategy)
- 在 `DrawCycloReducer.__init__` 中增加组件创建策略：
  - 若当前设计不允许新建组件，则直接使用 `design.rootComponent` 作为 `compReducer`。
  - 否则保留原有新建组件逻辑。
- 保持现有变量命名、缩进与注释风格，避免影响后续绘制流程。

## 5. 文件级变更清单 (File Changes)
- `[修改] CycloidalDrive.py`：在 `DrawCycloReducer.__init__` 中添加“rootComponent 直绘”分支。

## 6. 风险与验证
**风险**
- 在装配设计中分支判断不当可能导致组件未被创建。

**验证**
- 在零件设计文档运行脚本：应正常生成草图，不再报错。
- 在装配设计文档运行脚本：应仍可生成独立组件（或至少生成草图）。

## 7. 文档变更列表
- 无。
