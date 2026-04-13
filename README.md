# Legalize-CN

中国法律Git时间旅行仓库

本项目将中国法律文本转换为Git版本控制，实现法律演变的"时间旅行"功能。

## 项目起源

本仓库的初始提交时间设置为 **1949年9月21日**，
这一天是中国人民政治协商会议第一届全体会议通过《共同纲领》的日子，
象征着新中国法律体系的起点。

## 使用方法

```bash
# 克隆仓库
git clone https://github.com/legalize-cn/legalize-cn.git

# 查看某部法律的历史演变
git log --follow laws/刑法.md

# "穿越"到特定时间点查看法律状态
git checkout --detach `git log --format=%H --before="1997-10-01" laws/刑法.md | head -1`

# 比较两个版本之间的差异
git diff 1997版刑法 2015版刑法
```

## 数据来源

所有法律文本均来自官方数据源：
- 国家法律法规数据库 (flk.npc.gov.cn)
- 中国政府网政策文件库 (www.gov.cn)

## 许可证

- 代码：MIT License
- 法律文本：Public Domain（公共领域）

---

*本仓库由 Legalize-CN 自动化流水线生成*
