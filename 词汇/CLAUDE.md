# 词汇背诵项目

## 数据来源
- `墨墨单词本-单词之间.pdf`：墨墨背单词 App 导出的考研核心词，共 40 天，按词根词缀组织
- `高频词.pdf`：启航"单词之间"高频词（与墨墨单词本内容相同）

## 学习顺序
1. **先背高频词**（墨墨单词本 Daily 01-40），基础词和低频词暂不考虑
2. 按 PDF 中以天为单位，逐天消化

## 笔记格式规范

每天生成一个文件：`笔记/DayXX-主题.md`

### 文件结构
```
# Day XX 主题

> 📊 X 个词根组 · X 个单词

## 词根一：-xxx-（中文含义）

### 📖 词根故事（通俗来源/联想/为什么这个意思）

### 🔗 单词家族（一张表全记住）
| 单词 | 拆解 | 释义 | 记忆线索 |

### 🧩 记忆口诀（可选，单词多时用）

### 📝 必背例句（每词根 1-2 句最核心的）

## 📋 Day XX 自测清单
每个词根组对应一组自测题：
- 提示 → 写出单词（英汉互译）
- 翻译题（必背句子）
```

### 约束
- 必须覆盖 PDF 中当天的**全部单词**，不允许遗漏
- 词根数量按 PDF 实际分组合并
- 自测清单要能检验每个单词的掌握情况
- 中文解释要通俗，避免术语堆砌
- 词根故事优先讲"为什么是这个意思"，能联想记忆最好

### 创建流程（必须遵守）
1. **先提取**：用 Python 从 PDF 提取当天全部单词（含编号、拼写、音标、词性）
2. **创建笔记**：按格式规范写出完整笔记
3. **审核检查**：逐项核对以下清单，**全部通过才算完成**（详见下方审核清单）
4. **核验表头**：`> 📊 X 个词根组 · X 个单词` 中的数字必须与实际一致

### 审核清单（创建完成后必须逐项检查）

| # | 检查项 | 通过标准 |
|---|--------|------|
| 1 | **单词完整性** | 运行提取脚本，笔记覆盖 PDF 中当天**全部单词**，0 遗漏 |
| 2 | **音标完整性** | 每个 `**word**` 必须紧跟 `/phonetic/`，不允许存在无音标的词条 |
| 3 | **词根组数量** | 表头 `📊 X 个词根组` 的 X 与实际分组数一致 |
| 4 | **大组拆分** | 10+ 词的词根组必须拆成子分类，每组 ≤8 词 |
| 5 | **自测覆盖率** | 运行覆盖率脚本，自测提示条数 ≥ 表格单词数的 **70%** |
| 6 | **熟词僻义** | 有熟词僻义的单词标注 ★ |
| 7 | **拼写变体处理** | -ise/-ize 等变体在记忆线索栏标注，不重复计为独立词条 |
| 8 | **非成组词分组** | 非成组词按主题（身体/心理/商业/科技…）二次分组，每组 3-5 词 |

**审核不通过 → 回到步骤 2 修改 → 重新审核 → 直到全部通过。**

### 审核脚本
```python
# 在笔记创建完成后运行此脚本，检查全部 8 项审核清单
import re, os

fname = '笔记/DayXX-高频词根.md'
day = 'XX'

# ====== PDF 单词列表（从提取结果填入） ======
# 每次创建新 Day 时，先从 PDF 提取当天全部单词替换此列表
pdf_words = []  # ← 替换为提取到的单词列表

# 拼写变体白名单（-ise/-ize 等，不算遗漏）
spelling_variants = {'authorise','unauthorised','authorisation','organise',
    'centralise','civilise','agonise','colour','honour','labour'}

with open(fname) as f: content = f.read()

# ========== [1] 单词完整性 ==========
table_w = set(re.findall(r'\|\s*\*\*([a-zA-Z\-\s]+)\*\*\s*/', content))
table_w = {w.strip().lower().replace(' ','') for w in table_w if len(w.strip())>2}
missing = [w for w in pdf_words if w not in table_w]
real_missing = [w for w in missing if w not in spelling_variants]
print(f"[1] 单词完整性: PDF {len(pdf_words)}→表格 {len(table_w)}, 遗漏 {len(real_missing)}")
if real_missing: print(f"    ❌ 遗漏: {real_missing}")
if [w for w in missing if w in spelling_variants]:
    print(f"    拼写变体(不重计): {[w for w in missing if w in spelling_variants]}")

# ========== [2] 音标完整性 ==========
all_bold = re.findall(r'\|\s*\*\*([a-zA-Z\-\s]+)\*\*\s*([^|]*)\|', content)
no_phon = [w.strip().lower().replace(' ','') for w,r in all_bold 
           if len(w.strip())>2 and not re.search(r'/[^/]{2,}/', r)]
pct_phon = (1 - len(no_phon)/len(table_w))*100 if table_w else 0
print(f"[2] 音标完整性: {len(table_w)-len(no_phon)}/{len(table_w)} = {pct_phon:.0f}%")
if no_phon: print(f"    ❌ 缺音标: {no_phon}")

# ========== [3] 词根组数量 ==========
h_groups = int(re.search(r'📊\s*(\d+)\s*个词根组', content).group(1)) if re.search(r'📊', content) else 0
actual_g = len(re.findall(r'^## 词根[一二三四五六七八九十]', content, re.M))
print(f"[3] 词根组数量: 表头 {h_groups} vs 实际 {actual_g} {'✅' if h_groups==actual_g else '❌'}")

# ========== [4] 大组拆分 ==========
issues = []
sections = re.split(r'^## 词根', content, flags=re.M)
for s in sections[1:]:
    t = s.split('\n')[0].strip()[:35]
    wc = len(re.findall(r'\*\*[a-zA-Z\-\s]+\*\*\s*/', s))
    sc = len(re.findall(r'### 🧩', s))
    # 检查每个子组是否 ≤8 词
    for st in re.findall(r'### 🧩[^\n]*\n\n((?:\|.*\|\n)+)', s):
        sub_wc = len(re.findall(r'\*\*[a-zA-Z\-\s]+\*\*\s*/', st))
        if sub_wc > 8: issues.append(f'{t}→子组{sub_wc}词>8')
    # 非成组词不算大组
    if wc >= 10 and sc == 0 and '非成组' not in t:
        issues.append(f'{t}({wc}词无子分类)')
print(f"[4] 大组拆分: {'✅ 通过' if not issues else '❌ ' + ', '.join(issues)}")

# ========== [5] 自测覆盖率 ==========
ts = content.find('自测清单')
test_prompts = re.findall(r'\|\s*([^|*_\n]{2,40})\s*\|\s*_{3,}', content[ts:]) if ts>0 else []
pct_test = len(test_prompts)/len(table_w)*100 if table_w else 0
print(f"[5] 自测覆盖率: {len(test_prompts)}条/{len(table_w)}词 = {pct_test:.0f}% {'✅' if pct_test>=70 else '❌ 需≥70%'}")

# ========== [6] 熟词僻义 ==========
stars = len(re.findall(r'★', content))
print(f"[6] 熟词僻义: ★ {stars} 处 {'✅' if stars>0 else '❌ 未标注'}")

# ========== [7] 拼写变体 ==========
var_count = len(re.findall(r'(?:authoris|organis|centralis|civilis|agonis)', content))
p7 = var_count > 0
print(f"[7] 拼写变体: 标注 {var_count} 处 {'✅ 已处理' if p7 else '⚠️ 检查是否有拼写变体'}")

# ========== [8] 非成组词分组 ==========
ng_pos = content.find('非成组词')
p8 = True  # 默认为 N/A（全部归入词根组时通过）
if ng_pos > 0:
    ng_section = content[ng_pos:]
    # 检测主题分组标记
    themes = re.findall(r'\*\*[①②③④⑤⑥⑦⑧⑨⑩]', ng_section)
    # 检测每个主题组下的单词数量（从主题标记到下一个主题标记之间）
    theme_word_counts = []
    for i, marker in enumerate(themes):
        start = ng_section.find(f'**{marker}**')
        end = ng_section.find(f'**', start+5) if i < len(themes)-1 else len(ng_section)
        segment = ng_section[start:end]
        wc = len(re.findall(r'\*\*[a-zA-Z\-\s]+\*\*\s*/', segment))
        theme_word_counts.append((marker, wc))
    # 验证每组 3-5 词
    bad_themes = [(m, wc) for m, wc in theme_word_counts if wc < 3 or wc > 5]
    print(f"[8] 非成组词: {len(themes)}主题组, 每组词数: {[wc for _,wc in theme_word_counts]}")
    if bad_themes:
        print(f"    ❌ 不合规(需3-5词/组): {bad_themes}")
        p8 = False
    elif len(themes) == 0:
        print(f"    ❌ 未按主题分组")
        p8 = False
    else:
        print(f"    ✅ 通过")
else:
    print(f"[8] 非成组词: N/A（全部归入词根组）✅")
    p8 = True

# ========== 总结 ==========
checks = {
    '[1] 单词完整性': len(real_missing)==0,
    '[2] 音标完整性': len(no_phon)==0,
    '[3] 词根组数量': h_groups==actual_g and h_groups>0,
    '[4] 大组拆分': len(issues)==0,
    '[5] 自测覆盖率': pct_test>=70,
    '[6] 熟词僻义': stars>0,
    '[7] 拼写变体': p7,
    '[8] 非成组词': p8,
}
passed = sum(checks.values())
failed_items = [k for k,v in checks.items() if not v]
print(f"\n{'='*40}")
print(f"审核结果: {passed}/8 项通过")
if failed_items:
    print(f"❌ 未通过: {', '.join(failed_items)}")
else:
    print(f"✅ 全部通过!")
```

### 音标规则
- 每个单词**必须标注音标**，格式：**word** /phonetic/
- 音标从 PDF 提取，不可省略

### 归纳记忆要求
- 大词根组（10+ 词）**必须拆分成子分类**（如：基础链、真实链、互动链、代理人链）
- 子分类用小标题标注，每组 3-5 个词，方便分批记忆
- 5 个以上单词的词根组**尽量配记忆口诀**（押韵顺口溜）
- **熟词僻义**用 ★ 标注
- 拼写变体（-ise/-ize, -isation/-ization）在记忆线索栏标注，不单独列词条

### 常见陷阱
- PDF 提取可能混入相邻天的单词，必须核对单词编号
- 拼写变体（authorise/authorize）算同一个词，不重复计数
- PDF 内容按页眉 `墨墨背单词2027考研英语单词之间 ■Day XX` 分天，同一 Day 可能跨多页
- 单词编号连续跨天递增（Day01: 1-74, Day02: 75-148, ...），提取时按编号范围分段

### 自测清单质量标准
- **覆盖率要求**：自测提示条数 ≥ 单词表中词数的 **70%**
- 自测提示用**中文**（如"加速(v.)"、"有能力的"），不要用英文单词本身
- 自测按**同一子分类**编排，与正文结构一一对应
- 每做完一 Day 必须跑 Python 覆盖率脚本验证

### 非成组词处理
- 不隶属于任何词根组的单词统一归入"非成组词 — X 字母"
- 非成组词**按主题二次分组**（如：身体/健康、心理/态度、金钱/商业、科技/算法）
- 每组 3-5 个词，避免堆成一张大表

### 创建后自检脚本
```python
# 跑这个验证覆盖率和遗漏
import re
with open('笔记/DayXX-高频词根.md') as f: content = f.read()
table_w = set(re.findall(r'\|\s*\*\*([a-zA-Z\-\s]+)\*\*\s*/', content))
test_p = re.findall(r'\|\s*([^|*_\n]{2,40})\s*\|\s*_{3,}', content[content.find('自测清单'):])
print(f'{len(table_w)} words → {len(test_p)} prompts = {len(test_p)/len(table_w)*100:.0f}%')
```

## 复习建议
- 每次只记一个**子分类**（3-5 词），不要试图一次记整个词根组
- 先读词根故事 → 看单词矩阵 → 遮右栏自测 → 错词标记
- 每天学新内容前，先做前一日的自测（间隔复习）
- 重点掌握：熟词僻义（★标注）> 完全生词 > 已会单词
