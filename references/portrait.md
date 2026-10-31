# 画像数据更新规则（portrait.md）

画像展示页模板为 `assets/portrait-wall.html`。每次察觉后，把本次结果写入页内 `DATA` 对象并重新交付页面。本文件定义数据结构、加权与徽章规则，保证"每次察觉都正确累积"。

## DATA 结构

```js
var DATA = {
  streak: 连续察觉天数（int）,
  total: 累计察觉次数（int）,
  lastDate: "YYYY-MM-DD"（最近一次察觉日期）,
  dims: { mode:0, attr:0, emo:0, truth:0, action:0, energy:0 },  // 六维进度，0-100
  cards: [ { date:"YYYY-MM-DD", emotion:"情绪名", event:"事件一句话", core:"核心察觉", l1:"心理层", l2:"社经层", l3:"历史层", action:"纠正动作" } ],
  badges: { first:false, s3:false, s7:false, n10:false, n30:false, full:false }
};
```

## 六维含义

| key | 维度 | 含义 |
|---|---|---|
| mode | 模式识别 | 认出自己的行为/思维模式 |
| attr | 归因校准 | 分清"我的锅/结构的锅/阶段的锅" |
| emo | 情绪察觉 | 能命名情绪、看见回路 |
| truth | 真话勇气 | 接受并说出不舒服的真话 |
| action | 行动落地 | 纠正动作真正执行 |
| energy | 能量管理 | 减少内耗、守住精力 |

## 每次察觉的加权规则（单次上限 100）

基础分：
- 有 mode（认出模式）+14
- 有 truth（接受一句真话）+12
- 有 emo（命名情绪）+10
- 有 energy（识别内耗/能量泄漏）+8

层数加分（写全三层说明拆解得深）：
- 有 l1（心理层）→ mode +8
- 有 l2（社经层）→ attr +16
- 有 l3（历史层）→ attr +10、energy +6

动作加分：
- 有 action（可验证动作）→ action +18

执行：对 `DATA.dims` 各维度按本次应加分累加，单维封顶 100。

## 连续天数（streak）算法

- 若 `lastDate` 为空：streak = 1。
- 若本次日期 = lastDate 前一天：streak += 1。
- 若本次日期 = lastDate 当天：streak 不变（同日不重复累计）。
- 否则（断档）：streak = 1。
- 完成后 `lastDate` = 本次日期，`total += 1`（同日重复察觉不增加 total 和 streak，只追加卡）。

## 徽章条件（每次察觉后重算）

| id | 条件 |
|---|---|
| first | total ≥ 1 |
| s3 | streak ≥ 3 |
| s7 | streak ≥ 7 |
| n10 | total ≥ 10 |
| n30 | total ≥ 30 |
| full | 六维全部 = 100 |

## 段位边界（由画像完整度 = 六维均值决定）

| 均值 | 段位 |
|---|---|
| 0-24 | 新手观察者 |
| 25-49 | 初级察觉者 |
| 50-74 | 资深拆解者 |
| 75-99 | 上帝视角持有者 |
| 100 | 星图绘制者 |

## 情绪字段取值

平静 / 开心 / 焦虑 / 烦躁 / 委屈 / 疲惫 / 愧疚 / 期待（页面内置 8 色映射，取这 8 个之一）。

## 更新流程

1. 按加权规则算出本次各维加分。
2. 追加 `cards` 新卡（最新在前渲染，页面会自动倒序）。
3. 更新 streak / total / lastDate / dims / badges。
4. 打开模板 `assets/portrait-wall.html` 校对渲染（有截图自检脚本则跑一遍），交付更新后的 HTML。
