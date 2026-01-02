# 战斗系统架构

## 重构后的设计

系统现在采用**配置驱动**的方式，所有的技能效果、伤害、恢复数值都在配置中定义，而不是硬编码在代码中。

## 核心配置

### 1. 敌人配置 (setup.Enemies)
```javascript
slime: {
    name: "史莱姆",
    hp: 5,
    maxHp: 5,
    damage: 1,           // 敌人的攻击伤害
    speed: 5,            // 行动速度
    description: "..."
}
```

**添加新敌人：**
- 在 setup.Enemies 中添加新的敌人对象
- 设置名称、HP、伤害、速度等属性
- 伤害值将在执行敌人回合时使用

### 2. 技能效果配置 (setup.DiceEffects)
```javascript
setup.DiceEffects = {
    attack: {
        name: "攻击",
        damage: 1              // 攻击伤害
    },
    special_lifesteal: {
        name: "狂野冲刺",
        damage: 6,             // 伤害值
        heal: 3                // 恢复值
    },
    special_heal: {
        name: "冥想",
        heal: 5,               // 恢复值
        damageReduction: 0.5   // 伤害减免比例(0-1)
    },
    special_block: {
        name: "刀锋旋舞",
        perfectBlock: true     // 是否完全抵挡
    },
    defend: {
        name: "防御",
        teamShield: 1          // 获得的护盾
    },
    empty: {
        name: "空白"           // 无效果
    }
}
```

**添加新技能：**
- 在 setup.DiceEffects 中添加新的技能对象
- 定义技能的名称和相关数值（伤害、恢复、效果值等）
- 在英雄的 dice 数组中添加该技能代码
- 代码会自动识别并应用技能效果

### 3. 英雄配置 (setup.Heroes)
```javascript
yiDaShi: {
    name: "易大师",
    hp: 100,
    maxHp: 100,
    speed: 10,
    damageReduction: 0,
    dice: ["attack", "attack", "special_heal", "defend", "empty", "empty"]
}
```

**添加新英雄：**
- 在 setup.Heroes 中添加新的英雄对象
- 设置名称、HP、速度等基础属性
- dice 数组定义英雄的骰子面（可用的技能）
- 技能代码必须在 setup.DiceEffects 中定义

## 执行流程

### 1. 骰子投掷阶段
- 掷出随机骰子（从英雄的 dice 数组中选择）
- 保存到 `$battle.currentDice`

### 2. 行动选择阶段
- 如果是需要目标的技能（attack、special_lifesteal），进入**目标选择** passage
- 如果是自动技能（defend、special_heal、special_block、empty），进入**执行行动** passage

### 3. 执行阶段
```
// 步骤1：查找技能配置
_effect = setup.DiceEffects[$battle.currentDice]

// 步骤2：根据技能类型执行相应效果
if (attack) → 造成伤害
if (defend) → 增加护盾
if (special_heal) → 恢复HP并设置减伤
if (special_block) → 设置完全抵挡
if (special_lifesteal) → 造成伤害并恢复HP
```

## 易于扩展的改进

### 添加新伤害值
```javascript
setup.DiceEffects = {
    attack: {
        damage: 2  // 改为2点伤害
    }
}
```

### 添加新敌人
```javascript
setup.Enemies = {
    goblin: {
        name: "地精",
        hp: 10,
        maxHp: 10,
        damage: 2,
        speed: 7,
        description: "贪心的地精！"
    }
}
```

### 添加新技能
1. 在 setup.DiceEffects 中定义效果
2. 在英雄的 dice 数组中添加该技能
3. 如果是目标技能，在目标选择中添加对应按钮

## 优势

✓ **配置集中** - 所有数值在一个地方定义
✓ **易于维护** - 修改数值无需改代码逻辑
✓ **易于扩展** - 添加新敌人/技能只需修改配置
✓ **代码复用** - 执行逻辑通用，不需要为每个技能写特殊代码
✓ **数值平衡** - 修改数值只需改配置，方便做游戏平衡测试
