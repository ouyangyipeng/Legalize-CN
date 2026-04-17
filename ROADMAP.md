# Legalize-CN 项目路线图

> 本文档记录项目的设计思路、决策过程和每次迭代的变更点。

## 项目概述

**Legalize-CN** 是一个将中国现行有效法律法规转化为 Git 仓库的开源项目。核心价值在于：
- 通过 Git Diff 实现法律变迁的极速对比
- 通过 Git Checkout 实现历史版本的时间旅行
- 为法律科技应用提供高质量的结构化数据底座

## 技术决策记录

### 2026-03-29 项目启动

#### 决策1：数据源选择
- **首选数据源**：国家法律法规数据库 (https://flk.npc.gov.cn/)
  - 理由：中国人大网主办，最权威，涵盖宪法、法律、行政法规
  - 优势：文本质量高，元数据完整
- **备选数据源**：中国政府网政策文件库
  - 用途：补充行政法规和部门规章

#### 决策2：技术栈选型
| 组件 | 技术选择 | 理由 |
|------|----------|------|
| 爬虫框架 | Playwright + httpx | 动态渲染支持 + 高性能异步 |
| HTML解析 | BeautifulSoup4 + lxml | 鲁棒性强，容错能力好 |
| Git操作 | GitPython | Python原生，底层命令支持完整 |
| 配置管理 | Pydantic + YAML | 类型安全，配置可验证 |
| 日志系统 | loguru | 开箱即用，结构化日志 |

#### 决策3：Markdown层级映射标准
根据中国法律体系结构，确定如下映射规则：
```
编 (Part)     -> # H1
章 (Chapter)  -> ## H2
节 (Section)  -> ### H3
条 (Article)  -> #### H4 (统一使用H4，保持一致性)
款 (Paragraph) -> 自然段落，无标记
项 (Item)     -> 有序列表 (一)、(二)、(三)
目 (Sub-item) -> 数字列表 1. 2. 3.
```

#### 决策4：文件命名规范（已修订）
- **使用中文原名命名**：`中华人民共和国刑法.md`（而非拼音 `xing_fa.md`）
- 修订理由：现代Git/OS对UTF-8文件名支持已成熟；中文命名更具自文档性和可读性；Legalize-es也使用西班牙语命名
- 超长标题使用hash截断策略（>80字符时截断并添加MD5后缀）
- 发文字号命名方案（`npc-1997-83.md`）作为备选保留，但不再优先

#### 决策5：Git提交时间策略
- `GIT_AUTHOR_DATE` = 法律/修正案的官方发布日期

#### 决策6：YAML Frontmatter标准格式（2026-04-16修订）
统一字段名和值格式，消除两套格式并存的问题：
```yaml
title: "中华人民共和国刑法"
level: "法律"           # 效力层级（不再使用type）
status: "现行有效"      # 不再使用"有效"或数字码
publish_date: "1997-03-14"  # 纯日期（不再带时间戳）
authority: "全国人民代表大会"  # 发布机关（不再使用office）
doc_id: "主席令第83号"
source_url: "https://flk.npc.gov.cn/..."
```

#### 决策7：目录分类规范（2026-04-16修订）
按效力层级严格分离目录，消除混杂：
```
constitution/       → 宪法（1部）
laws/               → 法律（547部）
regulations/        → 行政法规（636部）
judicial/           → 司法解释（677部）
local_regulations/  → 地方性法规（15224部）
department_rules/   → 部门规章（预留）
others/             → 修改/废止决定等（644部）
```

#### 决策8：Markdown层级映射标准（2026-04-16修订）
条（Article）统一使用 `#### H4` 标题标记，确保Git diff精确显示单条变更：
- 移除目录区域重复章节标题（GitHub自动从标题生成目录）
- 移除全角空格缩进（Markdown通过空行分隔段落）
- 移除正文冗余元数据行（**发布机关**等已在YAML中）

## 项目进度

### 2026-03-29 MVP完成

#### 已完成模块

1. **项目结构初始化** ✅
   - 目录结构：legalize-builder/, legalize-cn/, data_cache/, logs/
   - 配置文件：pyproject.toml, requirements.txt, .env.example

2. **爬虫模块** ✅
   - `scrapers/base_scraper.py` - 基础爬虫类（缓存、速率限制、UA轮换）
   - `scrapers/npc_scraper.py` - 国家法律法规数据库爬虫
   - `scrapers/gov_scraper.py` - 中国政府网爬虫
   - `scrapers/playwright_scraper.py` - SPA动态渲染爬虫

3. **解析模块** ✅
   - `parsers/text_normalizer.py` - 文本归一化（全角空格、Unicode、BOM）
   - `parsers/structure_parser.py` - 法律结构解析（编章节条款项目）
   - `parsers/metadata_extractor.py` - 元数据提取与验证
   - `parsers/markdown_generator.py` - Markdown + YAML生成

4. **Git引擎** ✅
   - `git_engine/repo_manager.py` - 仓库管理
   - `git_engine/commit_builder.py` - 提交构建
   - `git_engine/time_travel.py` - 时间旅行引擎（伪造提交时间）
   - `git_engine/diff_analyzer.py` - 差异分析

5. **流水线** ✅
   - `main_pipeline.py` - 完整流水线编排
   - 支持 --dry-run 测试模式

6. **CI/CD** ✅
   - `.github/workflows/update-laws.yml` - 自动更新工作流
   - `README.md` - 项目文档
   - `CONTRIBUTING.md` - 贡献指南

#### 核心模块测试结果

```
[OK] TextNormalizer: 全角空格已转换
[OK] ChineseNumberConverter: 中文数字转换正确
[OK] MarkdownGenerator: YAML生成正确
[OK] DiffAnalyzer: 差异分析正确
[OK] Git Time-Travel: 提交日期正确伪造（1979-07-01, 1997-03-14, 2020-05-28）
```

#### 示例数据流水线运行结果

```
处理: 中华人民共和国刑法 (1979-07-01) - 3条
处理: 中华人民共和国刑法 (1997-03-14) - 4条
处理: 中华人民共和国民法典 (2020-05-28) - 4条

Git历史（按时间排序）:
2020-05-28 :recycle: 修订: 中华人民共和国民法典
1997-03-14 :recycle: 修订: 中华人民共和国刑法
1979-07-01 :recycle: 发布: 中华人民共和国刑法
```

#### 待完成

- [ ] RAW_DATA_COLLECT - 实际数据收集（需稳定网络连接，被反爬虫阻止）
- [x] Git仓库初始化与首次提交（使用示例数据完成）
- [ ] 边缘情况处理与V1.0锁定

#### 技术发现

1. **国家法律法规数据库是SPA应用**
   - 需要Playwright进行动态渲染
   - API端点存在但需要正确的请求参数

2. **中文数字转换**
   - 支持：一~百的转换
   - 需要处理：十一~十九、二十一~九十九等复合数字

3. **Unicode处理**
   - 全角空格 \u3000 需转换为普通空格
   - BOM字符 \ufeff 需移除
   - HTML实体需正确解码

4. **Git时间旅行**
   - Git接受 `@timestamp +timezone` 格式（如 `@299606400 +0800`）
   - `GIT_AUTHOR_DATE` = 法律发布日期
   - `GIT_COMMITTER_DATE` = 同上
   - 确保历史线性：按 `publish_date` 升序排序后提交

5. **StructureParser**
   - 使用 `normalize()` 而非 `normalize_for_matching()` 以保留换行符
   - 正则表达式依赖行首匹配（`^`），换行符必须保留

## 阶段进度追踪

### 阶段组A：需求与基建
- [x] 需求分析与文档阅读
- [ ] PROJECT_INIT
- [ ] ARCHITECTURE_DESIGN

### 阶段组B：数据源突破与抓取
- [ ] DATA_SOURCE_MAPPING
- [ ] SCRAPER_BUILD
- [ ] ANTI_SPIDER_BYPASS [门控]
- [ ] RAW_DATA_COLLECT

### 阶段组C：文本解析与结构化
- [ ] REGEX_ENGINE_DESIGN
- [ ] HTML/PDF_TO_MD
- [ ] YAML_METADATA_INJECT

### 阶段组D：Git引擎设计
- [ ] GIT_OPERATOR_DEV
- [ ] DIFF_OPTIMIZATION

### 阶段组E：自动化流水线集成
- [ ] PIPELINE_ASSEMBLY
- [ ] DRY_RUN_TEST

### 阶段组F：异常分析与自愈
- [ ] ERROR_ANALYSIS
- [ ] PIPELINE_REFINE

### 阶段组G：Git历史引擎构建
- [ ] GIT_REPO_INIT
- [ ] TIMELINE_MOCK
- [ ] BATCH_COMMIT_EXECUTION

### 阶段组H：CI/CD与开源准备
- [ ] GITHUB_ACTIONS_WRITE
- [ ] QUALITY_GATE [门控]
- [ ] DOCS_GENERATION
- [ ] OPEN_SOURCE_RELEASE

### 阶段组I：迭代与防御
- [ ] EDGE_CASE_PATCH
- [ ] COMMUNITY_SIMULATION
- [ ] V1.0_LOCK

## 风险与问题记录

| 日期 | 风险/问题 | 影响 | 缓解措施 | 状态 |
|------|-----------|------|----------|------|
| - | - | - | - | - |

## 变更日志

### v0.10.0 (2026-04-17)
- **民法典修正案历史重建完成 — 全部优先级法律P0-P3+民法典全覆盖**
  - 创建 [`rebuild_civil_code_history.py`](legalize-builder/scripts/rebuild_civil_code_history.py) — 民法典专用处理脚本
  - 补充婚姻法1950年版（新中国第一部法律！），Wikisource繁体标题 `中華人民共和國婚姻法 (1950年)`
  - 婚姻法完整历史重建：1950→1980→2001→废止(被民法典取代)
  - 民法通则/收养法添加"废止commit"（2021-01-01，被民法典废止）
  - 6部单版本前身法律frontmatter更新（abolish_date/replaced_by）
  - 民法典frontmatter添加 `predecessor_laws` 元数据（9部前身法律）
  - 修复涉外民事关系法律适用法status错误（现行有效→已废止）
  - 关键技术发现：Git 2.34.1不支持1970年前日期（pre-epoch），婚姻法1950年版git日期设为1970-01-01，真实日期1950-04-13记录在frontmatter中
  - Wikisource API rate-limit问题：403 "Please respect our robot policy"，需要缓存机制避免重复请求

### v0.9.0 (2026-04-16)
- **P3批量修正案历史重建完成 — 193部法律全部完成**
  - 创建 [`auto_discover_law_versions.py`](legalize-builder/scripts/auto_discover_law_versions.py) — 自动发现Wikisource上的多版本法律
    - 使用Wikisource搜索API `action=query&list=search` 自动发现187部多版本法律
    - 773个版本页面被发现，其中172部实际法律（排除决定/解释）726个版本页面
  - 创建 [`auto_pipeline_p3.py`](legalize-builder/scripts/auto_pipeline_p3.py) — 全自动流水线（fetch + format + rebuild）
    - 自动从Wikisource文本中提取日期（`detect_dates_from_text()`），无需手动配置LAW_DATES
    - 缓存formatted versions到 `data_cache/law_versions_p3/`，支持断点续跑
    - 错误恢复机制：stash + checkout + reset + checkout--，确保Git状态始终干净
    - 结果：172 rebuilt, 0 skipped, 0 errors
  - 技术改进：
    - 修复 `re.DOTALL` vs `re.RE.DOTALL` bug
    - 修复 `strptime(None)` 崩溃 — 使用 `pub_date or f"{year}-01-01"` 兜底
    - 增强Git错误恢复 — 海关法strptime错误导致后续全部失败，改进stash/cleanup逻辑后0错误完成
  - 累计统计：
    - 总提交数：26,352（history-rebuild分支）
    - 有修正案历史的法律：193部（含宪法+刑法+P0-P2 20部+P3 172部）
    - 304个merge提交（每部法律一个--no-ff merge）

### v0.8.0 (2026-04-16)
- **20部重要法律修正案历史重建完成（P0-P2全覆盖）**
  - Phase 1: Wikisource数据获取 — [`fetch_law_versions.py`](legalize-builder/scripts/fetch_law_versions.py)，84/85版本页面获取成功（99%成功率）
  - Phase 2: 文本格式化 — [`format_law_versions.py`](legalize-builder/scripts/format_law_versions.py)，84/84版本全部格式化完成
  - Phase 3: Git历史重建 — [`rebuild_law_history.py`](legalize-builder/scripts/rebuild_law_history.py)，20部法律84版本全部提交成功
  - 关键技术发现：
    - Wikisource命名格式：半角括号+空格 `中华人民共和国XXX (YYYY年)`，非全角`（YYYY年）`
    - Wikisource API：`action=query&list=categorymembers`返回403，必须使用`action=parse`
    - Python subprocess日期传递：`env=dict`方式导致Git日期解析失败，必须通过`os.environ`直接设置让子进程继承
    - Git内部日期格式：`%Y-%m-%d %H:%M:%S +0000`（非ISO 8601的T分隔格式）
  - 重建详情：
    - 刑事诉讼法(P0): 1979→1996→2012→2018 (4版本)
    - 公司法(P1): 1993→1999→2004→2005→2013→2018 (6版本)
    - 专利法(P1): 1984→1992→2000→2008→2020 (5版本)
    - 证券法(P1): 1998→2005→2014→2019 (4版本)
    - 商标法(P1): 1982→1993→2001→2013→2019 (5版本)
    - 个人所得税法(P1): 1980→1993→1999→2005→2011→2018 (6版本，2007年版本Wikisource不存在)
    - 著作权法(P1): 1990→2001→2010→2020 (4版本)
    - 药品管理法(P2): 1984→2001→2019 (3版本)
    - 保险法(P2): 1995→2002→2009→2014→2015 (5版本)
    - 行政处罚法(P2): 1996→2009→2017→2021 (4版本)
    - 选举法(P2): 1979→1982→1986→1995→2004→2010→2015 (7版本)
    - 土地管理法(P2): 1986→1988→1998→2004→2019 (5版本)
    - 劳动法(P2): 1994 (1版本)
    - 劳动合同法(P2): 2007→2012 (2版本)
    - 环境保护法(P2): 1989→2014 (2版本)
    - 食品安全法(P2): 2009→2015→2021 (3版本)
    - 兵役法(P2): 1984→1998→2009→2011→2021 (5版本)
    - 国家赔偿法(P2): 1994→2010 (2版本)
    - 消防法(P2): 1998→2008→2021 (3版本)
    - 道路交通安全法(P2): 2003→2007→2011→2021 (4版本)
  - 每部法律使用独立分支(`XXX-fix`)按时间顺序提交，然后`--no-ff`合并到`history-rebuild`
  - 修复了 `subprocess.run(env=dict)` 导致Git日期解析失败的bug，改为通过`os.environ`直接设置
  - 修复了 `subprocess.run(text=True)` 中文编码解码崩溃的bug，添加 `encoding="utf-8", errors="replace"`

### v0.7.0 (2026-04-16)
- **GitHub默认分支切换完成**
  - `history-rebuild` 设为GitHub默认主分支，原`master`保留为后备分支
  - 两个分支无共同祖先（完全独立历史），无法常规merge
- **README.md全面重写**
  - 反映项目真实状态：24,447个文件、24,463个提交
  - 添加宪法和刑法修正案历史对照表格
  - 添加修正案历史重建优先级路线图（P0-P3四级）
  - 添加完整的仓库目录结构说明
  - 添加实际可运行的git命令示例（替换原来的占位符）
  - README和CONTRIBUTING已复制到Git仓库根目录并提交
- **重要法律修正案优先级清单**
  - 普查155部重要法律的修订历史
  - 创建 [`AMENDMENT_HISTORY_PRIORITY.md`](plans/AMENDMENT_HISTORY_PRIORITY.md)
  - P0: 刑事诉讼法、民法典（需特殊处理）
  - P1: 公司法、证券法、专利法、商标法、个人所得税法等8部
  - P2: 行政处罚法、选举法、土地管理法等11部
  - P3: 其他155部重要法律（批量处理）

### v0.6.0 (2026-04-16)
- **刑法修正案历史重建完成**
  - 从Wikisource获取刑法历次修正版本：16个完整刑法文本 + 12个修正案 + 1个单行刑法（29/29页面，100%成功）
  - 编写 [`fetch_criminal_law_versions.py`](legalize-builder/scripts/fetch_criminal_law_versions.py) - Wikisource数据获取脚本
  - 编写 [`format_criminal_law_versions.py`](legalize-builder/scripts/format_criminal_law_versions.py) - 文本格式化脚本
  - 编写 [`rebuild_criminal_law_history.py`](legalize-builder/scripts/rebuild_criminal_law_history.py) - Git历史重建脚本
  - 刑法Git时间线：1997→1998→1999→2001/08→2001/12→2002→2005→2006→2009/02→2009/08→2011→2015→2017→2020→2023（15个版本）
  - 1979年刑法作为独立文件：`laws/中华人民共和国刑法（1979年）.md`
  - 修正案文件与刑法通过YAML `applies_to` 字段关联
  - 条文数从1997版452条逐步递增到2023版505条，符合预期
  - 合并criminal-law-fix分支到history-rebuild并推送GitHub
- **Wikisource数据源突破**
  - 发现Wikisource Template页面有刑法全部历次修正版本的完整文本
  - 使用MediaWiki API批量获取页面内容（parse action）
  - 数据缓存到 `data_cache/criminal_law_versions/`

### v0.5.0 (2026-04-16)
- **设计文档与实现对齐完成**
  - 创建 [`DESIGN_ALIGNMENT_PLAN.md`](plans/DESIGN_ALIGNMENT_PLAN.md) 记录6大差异分析
  - YAML Frontmatter统一：type→level, office→authority, 状态值修复（33908文件）
  - Markdown层级修复：第X条→#### H4标记（471983条文）
  - 目录分类修正：地方性法规分离到local_regulations/（24178文件）
  - 删除9464个同名重复文件
  - 移除冗余元数据行、全角空格缩进、重复目录区域
  - 编写 [`unify_format.py`](legalize-builder/scripts/unify_format.py) 和 [`reclassify_files.py`](legalize-builder/scripts/reclassify_files.py)
- **决策变更记录**
  - 决策4修订：中文命名保留（而非拼音），理由是现代Git对UTF-8支持成熟
  - 决策6新增：YAML Frontmatter标准格式（消除两套格式并存）
  - 决策7新增：目录分类规范（按效力层级严格分离）
  - 决策8新增：Markdown层级映射标准（条→H4）

### v0.4.0 (2026-04-13)
- **修正案解析器V2完成**
  - 创建 [`amendment_parser_v2.py`](legalize-builder/scripts/amendment_parser_v2.py)
  - 支持13种修正案格式解析
  - 支持操作类型：修改、增加、删除、替换、修改款项、修改序言、修改章节
  - 中文数字转换器支持1-200范围
  - 测试验证：1988、1993年宪法修正案解析正确

### v0.3.0 (2026-04-13)
- **宪法修正历史重建完成**
  - 从Wikisource获取1988、1993、1999、2004年修正后的完整宪法文本
  - 使用constitution-fix分支重建宪法Git历史
  - 宪法现在有6个版本：1982→1988→1993→1999→2004→2018
  - Git diff可正确显示每次修正的具体条文变更
  - 例如：1988年修正案第10条（土地使用权）、第11条（私营经济）
- **处理未跟踪文件完成**
  - 新增5430个提交（从18993增至24423）
  - 修复中文数字日期解析（如"一九八五年四月十日"）
  - 修复多种日期格式提取
- 推送history-rebuild分支到GitHub

### v0.2.0 (2026-03-30)
- 批量导入完成：33810个Markdown文件
- Git时间旅行提交完成：26599个提交
- 时间范围：1973-05-30 至 2023-07-27
- Git历史重建：880条重要法律的多版本合并
- 宪法7个版本合并为单一文件

### v0.1.0 (2026-03-29)
- 项目初始化
- 完成需求分析与技术选型
- 创建项目路线图