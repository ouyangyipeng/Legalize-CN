# 贡献指南

感谢您对 Legalize-CN 项目的关注！本文档将帮助您参与项目贡献。

## 🎯 贡献方式

### 1. 报告问题

如果您发现以下问题，请提交 Issue：

- **法律文本错误**：如条款缺失、内容错误、格式问题
- **解析错误**：如正则表达式未能正确识别条款结构
- **元数据错误**：如发布日期、效力层级等信息不准确
- **功能建议**：如新增数据源、改进解析逻辑等

提交 Issue 时请包含：

1. 问题描述
2. 相关法律名称和URL
3. 预期结果与实际结果
4. 如有可能，提供修复建议

### 2. 修复问题

#### 环境准备

```bash
# 克隆仓库
git clone https://github.com/legalize-cn/legalize-cn.git
cd legalize-cn

# 安装依赖
cd legalize-builder
pip install -r requirements.txt

# 安装Playwright（用于SPA网站）
pip install playwright
playwright install chromium
```

#### 代码规范

- **Python版本**：3.11+
- **格式化**：使用 `ruff` 进行代码格式化和 lint
- **类型注解**：所有函数必须包含类型注解
- **文档**：复杂逻辑必须添加注释说明

#### 提交规范

遵循 Conventional Commits 规范：

```
:<gitmoji>: <type>(<scope>): <subject>

示例：
:sparkles: feat(scraper): add new data source support
:bug: fix(parser): handle full-width number conversion
:recycle: refactor(git): optimize commit message format
:memo: docs(readme): update quick start guide
```

### 3. 添加数据源

如需添加新的法律数据源，请：

1. 在 `scrapers/` 目录创建新爬虫类
2. 继承 `BaseScraper` 基类
3. 实现 `fetch_law_list()` 和 `fetch_law_detail()` 方法
4. 添加必要的 CSS 选择器和 URL 模式
5. 在 `main_pipeline.py` 中集成新爬虫

示例：

```python
from .base_scraper import BaseScraper

class NewSourceScraper(BaseScraper):
    BASE_URL = "https://new-source.gov.cn"
    
    async def fetch_law_list(self) -> list[str]:
        # 实现列表抓取逻辑
        pass
    
    async def fetch_law_detail(self, url: str) -> dict:
        # 实现详情抓取逻辑
        pass
```

### 4. 改进解析器

如需改进法律文本解析：

1. **正则表达式**：在 `parsers/structure_parser.py` 中修改
2. **文本归一化**：在 `parsers/text_normalizer.py` 中添加新规则
3. **元数据提取**：在 `parsers/metadata_extractor.py` 中扩展

注意：解析器修改必须经过充分测试，确保不破坏现有功能。

## 🧪 测试

### 运行测试

```bash
cd legalize-builder
pytest tests/ -v
```

### Dry Run 测试

```bash
python -m main_pipeline --dry-run
```

### 验证法律文件

```bash
python scripts/validate_laws.py
```

## 📋 核心约束

在贡献代码时，请严格遵守以下约束（来自 RESTRICTS.md）：

### ❌ 严禁行为

1. **禁止使用大模型生成法律文本**：法律正文必须 100% 从官方源精确提取
2. **禁止跳过解析失败**：解析失败必须抛出 `ParseError` 并停止运行
3. **禁止混用空格**：必须应用严格的文本归一化
4. **禁止无缓存抓取**：所有下载必须保存到 `data_cache` 目录
5. **禁止伪造时间**：每次提交必须将 `GIT_AUTHOR_DATE` 设置为法律发布日期

### ✅ 必须行为

1. 实现本地缓存机制
2. 添加速率限制（礼貌抓取）
3. 正确处理异常并记录日志
4. 验证元数据完整性
5. 遵循 Git 提交时间伪造规则

## 🔧 开发流程

1. **Fork** 本仓库
2. **Clone** 您的 Fork
3. **Branch** 创建功能分支 (`git checkout -b feature/your-feature`)
4. **Code** 编写代码并测试
5. **Commit** 提交代码（遵循提交规范）
6. **Push** 推送到您的 Fork
7. **PR** 创建 Pull Request

### PR 检查清单

- [ ] 代码通过 `ruff` 检查
- [ ] 测试全部通过
- [ ] 新功能有对应测试
- [ ] 文档已更新
- [ ] 提交消息符合规范
- [ ] 未违反核心约束

## 📞 联系方式

- **Issues**：[GitHub Issues](https://github.com/legalize-cn/legalize-cn/issues)
- **讨论**：[GitHub Discussions](https://github.com/legalize-cn/legalize-cn/discussions)

---

再次感谢您的贡献！🙏