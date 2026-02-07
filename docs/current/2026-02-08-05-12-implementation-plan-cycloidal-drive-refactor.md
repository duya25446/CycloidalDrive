# 摆线减速器生成器脚本重构 - 实施计划

## 1. 目标 / 非目标

### 目标
- 修复 Python 语法错误，使脚本能在新版 Fusion360 中运行
- 修正代码中的拼写错误
- 更新 manifest 版本信息
- 验证 pickle 序列化在新版 Python 中的兼容性
- 保持现有代码架构和风格不变

### 非目标
- 不重构代码架构（保留全局变量模式）
- 不修改数学算法（摆线曲线计算公式完全保留）
- 不改变 UI 布局和交互逻辑
- 不重命名变量和函数（除拼写错误外）

---

## 2. 关键资产保留 (Mandatory Technical Assets)

### 数学算法（必须原样保留）

```python
# 合成シンプソン法による近似積分
def compositeSimpson(func,upper,lower, splitNum):
    splitNum=int(splitNum)
    if splitNum&0b1:    #奇数
        splitNum += 1
    h = (upper-lower)/splitNum
    ysum = func(lower) + 4*func(lower+h) + func(upper)
    for i in range(2,splitNum)[::2]:
        ysum += 2*func(lower+i*h) + 4*func(lower+(i+1)*h)
    return h/3*(ysum)

# 二分法
def bisectionMethod(func, upper, lower, maxError, maxCalcTimes=100):
    maxError=abs(maxError)
    calcTimes=0
    while True:
        calcTimes+=1
        x = (upper+lower)/2.0
        if (0.0 < func(x)*func(upper)):#符号判定
            upper=x
        else:
            lower=x
        if (upper-lower<=maxError):
            return x
        elif (calcTimes==maxCalcTimes):
            return x

# ニュートン法と中心差分近似微分を使用
def numericalAnalysis(func, initialValue, maxError, maxCalcTimes=100):
    maxError = abs(maxError)
    calcTimes=0
    x = initialValue
    dfx = lambda x: (func(x*1.000001)-func(x*0.999999)) / (x*0.000002)
    while True:
        calcTimes+=1
        xn = x-func(x)/dfx(x)
        if abs(xn-x)<=maxError:
            return xn
        elif calcTimes==maxCalcTimes:
            return xn
        x = xn
```

### CycloidalReducer 类核心计算方法（必须原样保留）

```python
## x of trochoid curve
def fxa(self, p):
    return (self.rc+self.rm)*math.cos(p) - self.rd*math.cos((self.rc+self.rm)/self.rm*p)

## y of trochoid curve
def fya(self, p):
    return (self.rc+self.rm)*math.sin(p) - self.rd*math.sin((self.rc+self.rm)/self.rm*p)

## x of differential trochoid curve
def dfxa(self, p):
    return -(self.rc+self.rm)*math.sin(p) + ((self.rc+self.rm)/self.rm)*self.rd*math.sin((self.rc+self.rm)/self.rm*p)

## y of differential trochoid curve
def dfya(self, p):
    return (self.rc+self.rm)*math.cos(p) - ((self.rc+self.rm)/self.rm)*self.rd*math.cos((self.rc+self.rm)/self.rm*p)

## x of 2nd order differential trochoid curve
def ddfxa(self, p):
    return -(self.rc+self.rm)*math.cos(p) + ((self.rc+self.rm)/self.rm)**2 * self.rd*math.cos((self.rc+self.rm)/self.rm*p)

## y of 2nd order differential trochoid curve
def ddfya(self, p):
    return -(self.rc+self.rm)*math.sin(p) + ((self.rc+self.rm)/self.rm)**2 * self.rd*math.sin((self.rc+self.rm)/self.rm*p)
```

---

## 3. 现状摘要

### 文件结构
```
CycloidalDrive/
├── CycloidalDrive.py          # 主脚本 (947行)
├── CycloidalDrive.manifest    # 清单文件
├── README.md                   # 说明文档
├── image/                      # UI 图片资源
└── docs/FusionAPI/             # API 参考文档
```

### 代码现状

| 组件 | 行数 | 状态 | 说明 |
|------|------|------|------|
| 数学函数 (compositeSimpson 等) | 62-111 | ✓ 正常 | 数值计算核心逻辑 |
| CycloidalReducer 类 | 115-348 | ✓ 正常 | 摆线参数计算 |
| DrawCycloReducer 类 | 350-579 | ✓ 正常 | Fusion360 草图绘制 |
| inputsToParameter 函数 | 581-632 | 🔴 语法错误 | line 587 缺少逗号 |
| settingComandInputsItem 函数 | 634-681 | 🟡 拼写错误 | 函数名缺少 'm' |
| 事件处理器类 | 689-911 | ✓ 正常 | 命令事件处理 |
| run 函数 | 913-946 | ✓ 正常 | 入口函数 |

### 修改切入点

1. **Line 587**: namedtuple 字段定义缺少逗号
2. **Line 634**: 函数名拼写错误 `settingComandInputsItem` → `settingCommandInputsItem`
3. **Line 721**: 函数调用处需同步更新
4. **CycloidalDrive.manifest**: 版本号更新

---

## 4. 设计方案 (Design & Strategy)

### 4.1 修复策略

采用**最小化修改**策略，保持与现有代码风格完全一致：

#### 修复 1: namedtuple 语法错误 (Line 582-589)

**原代码**:
```python
drawingParam = namedtuple("DrawingParam",
                        ("ringPinNum", "ringPinDia", "ringPinPitchDia",
                            "eccentricAmount", "plotDotNum",
                            "troGearAroundHoleNum", "troGearAroundHoleDia", "troGearAroundHolePosDia",
                            "troGearCentorHoleDia",
                            "outDiskPinNum", "outDiskPinDia","outDiskPinPosDia"
                            "isDrawTrochoidalGear", "isDrawRingPin","isDrawCentorHole", "isDrawAroundHole","isDrawOutputDiskPin"
                        ))
```

**修复后**:
```python
drawingParam = namedtuple("DrawingParam",
                        ("ringPinNum", "ringPinDia", "ringPinPitchDia",
                            "eccentricAmount", "plotDotNum",
                            "troGearAroundHoleNum", "troGearAroundHoleDia", "troGearAroundHolePosDia",
                            "troGearCentorHoleDia",
                            "outDiskPinNum", "outDiskPinDia", "outDiskPinPosDia",
                            "isDrawTrochoidalGear", "isDrawRingPin", "isDrawCentorHole", "isDrawAroundHole", "isDrawOutputDiskPin"
                        ))
```

#### 修复 2: 函数名拼写错误

**修改位置**:
- Line 634: 函数定义 `def settingComandInputsItem(inputs):`
- Line 721: 函数调用 `settingComandInputsItem(inputs)`

**修改为**: `settingCommandInputsItem`（添加 'm'）

### 4.2 Manifest 更新

**原内容**:
```json
{
	"autodeskProduct": "Fusion360",
	"type": "script",
	"author": "woodenCariper",
	"version": "1.0.0",
	...
}
```

**更新后**:
```json
{
	"autodeskProduct": "Fusion360",
	"type": "script",
	"author": "woodenCariper",
	"version": "2.0.0",
	"description":{
		"":	"This script create sketchs for cycloidal drive - Updated for modern Fusion360"
	},
	...
}
```

### 4.3 Pickle 兼容性验证

在 `saveInputsValues` 和 `loadInputsValue` 函数中添加版本兼容性处理：

```python
# 在 saveInputsValues 中添加协议版本
binaryValue = pickle.dumps(value, protocol=pickle.HIGHEST_PROTOCOL)

# 在 loadInputsValue 中添加异常处理
try:
    value = pickle.loads(binaryValue)
except (pickle.UnpicklingError, EOFError, AttributeError) as e:
    # 兼容性处理：返回默认值或重新初始化
    return False
```

---

## 5. 文件级变更清单 (File Changes)

| 类型 | 文件路径 | 变更说明 |
|------|----------|----------|
| [修改] | `CycloidalDrive.py` | Line 587: 修复 namedtuple 语法错误（添加逗号） |
| [修改] | `CycloidalDrive.py` | Line 587: 统一引号风格（单引号改双引号，保持一致性） |
| [修改] | `CycloidalDrive.py` | Line 634: 函数重命名 `settingComandInputsItem` → `settingCommandInputsItem` |
| [修改] | `CycloidalDrive.py` | Line 721: 函数调用同步更新 |
| [修改] | `CycloidalDrive.py` | Line 736: 更新 pickle dumps 添加协议参数 |
| [修改] | `CycloidalDrive.py` | Line 745-754: 添加 pickle loads 异常处理 |
| [修改] | `CycloidalDrive.manifest` | 版本号 1.0.0 → 2.0.0 |
| [修改] | `CycloidalDrive.manifest` | 更新描述信息 |

---

## 6. 风险与验证

### 风险识别

| 风险 | 严重性 | 缓解措施 |
|------|--------|----------|
| namedtuple 字段顺序变化破坏现有代码 | 🟡 中 | 保持字段顺序完全一致，仅添加逗号 |
| pickle 反序列化失败 | 🟡 中 | 添加异常处理和回退逻辑 |
| 函数重命名影响外部调用 | 🟢 低 | 函数仅在内部调用，无外部依赖 |
| Fusion360 API 不兼容 | 🟡 中 | 已通过文档对比确认核心 API 兼容 |

### 验证计划

#### V1: 语法验证
```bash
python -m py_compile CycloidalDrive.py
```

#### V2: Pickle 功能验证
```python
# 测试用例：验证参数保存/加载
test_values = [True, False, 42, "test", 3.14]
for v in test_values:
    serialized = pickle.dumps(v, protocol=pickle.HIGHEST_PROTOCOL)
    deserialized = pickle.loads(serialized)
    assert v == deserialized
```

#### V3: Fusion360 集成测试（手动）
1. 在 Fusion360 中运行脚本
2. 打开命令对话框
3. 输入测试参数：
   - Raducation ratio: 10
   - Eccentric amount: 0.2 mm
   - Ring pin diameter: 1.0 mm
   - Ring pin pitch diameter: 8.0 mm
   - Cycloidal curve plot num: 6
4. 勾选所有可选参数
5. 点击 OK 生成草图
6. 验证三个草图正确创建：
   - Cycloidal gear（摆线齿轮）
   - Ring pins（针轮销）
   - Output disk pin（输出盘销）
7. 关闭并重新打开脚本，验证参数被正确保存和加载

---

## 7. 文档变更列表

| 文档 | 变更类型 | 说明 |
|------|----------|------|
| `README.md` | 更新 | 添加版本 2.0.0 更新说明 |
| `CHANGELOG.md` | 新增 | 记录本次修复的变更内容 |
