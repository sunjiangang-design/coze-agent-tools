# Coze Agent Tools

Coze（扣子）Agent 工具、技能和模板集合。

## 目录结构

```
├── skills/
│   └── weiliang-daily-report/    # 伟良振动筛日报生成技能
│       ├── SKILL.md               # 技能说明
│       └── weiliang_report.py     # 主脚本
├── templates/
│   └── aily-style-template.md    # 飞书aily风格文档模板
├── docs/                          # 文档
└── .gitignore
```

## 技能说明

### 伟良日报（weiliang-daily-report）

新乡伟良振动筛工厂每日异常报表自动生成。
- 输入：主计划表 + 外购物料到货跟踪表（+可选成品入库表）
- 输出：异常报表 + 安全库存未到货表 + 成品入库标记
- 支持关键词分类（欠下料/焊接/粘网/机加/采购）

### aily风格模板

飞书云文档aily风格Markdown模板，用于生成结构化报告文档。

## 使用

技能通过扣子（Coze）Agent平台加载使用，详见各技能目录下的SKILL.md。
