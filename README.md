# OJ Resetta — 题目搬运追踪

在各大 OJ（洛谷、Codeforces、AtCoder）浏览题目时，自动检测该题是否已搬运到自有 Hydro OJ，并显示一键跳转链接；在自有 Hydro OJ 浏览题目时自动显示题目来源。

## 工作原理

```
mappings.xlsx (WPS 编辑)
    ↓ git push
GitHub Actions 自动运行 build.py
    ↓
data.json + csv/ 部署到 GitHub Pages
    ↑ fetch (带缓存)
oj-tracker.user.js (Tampermonkey 油猴脚本)
    ↓
标签页标题前缀 + Tampermonkey 菜单
```

## 目录结构

```
oj-resetta/
├── .github/workflows/build.yml      # GitHub Actions 自动构建+部署
├── build.py                         # Excel → JSON + CSV 转换脚本
├── mappings.xlsx                    # 题目映射表（WPS / Excel 编辑，唯一源文件）
├── oj-tracker.user.js.example       # 油猴脚本模板（可入库供参考）
├── .gitignore
└── README.md
```

> `data.json` 和 `csv/` 由 Actions 生成后直接部署到 GitHub Pages，不入库。
> 通过 `https://conanforever22.github.io/oj-resetta/csv/luogu.csv` 等链接可直接查看 CSV。

## 一次性设置

### 1. GitHub 仓库

```bash
git clone git@github.com:conanforever22/oj-resetta.git
cd oj-resetta
```

### 2. 启用 GitHub Pages

Settings → Pages → Source: **Deploy from a branch** → 分支选 `gh-pages` → `/ (root)` → Save

（gh-pages 分支会在第一次 push 后由 Actions 自动创建）

### 3. 配置油猴脚本

1. 复制 `oj-tracker.user.js.example` → `oj-tracker.user.js`
2. 修改脚本开头的 `HYDRO_PROBLEM_URL` 为你的 Hydro 域名
3. 在 Tampermonkey 中新建脚本，粘贴内容，保存

### 4. 准备 Excel

确保 `mappings.xlsx` 包含以下工作表：

| 工作表名 | A 列（源题号） | B 列（自有题号） |
|---------|---------------|-----------------|
| 洛谷 | P1001 | 100 |
| Codeforces | cf2038a | 200 |
| AtCoder | abc123d | 300 |

- 第一行为表头（任意文字）
- 数据从第 2 行开始
- 题号格式见下方说明

## Excel 题号格式规范

| OJ | 来源 URL 示例 | Excel 中写法 | 说明 |
|----|-------------|-------------|------|
| 洛谷 | `luogu.com.cn/problem/P1001` | `P1001` | 直接写题号 |
| Codeforces | `codeforces.com/problemset/problem/2038/A` | `cf2038a` | `cf` + 题号 + 字母，小写 |
| CF hard version | `.../problem/2038/C1` | `cf2038c1` | `cf` + 题号 + 字母 + 数字 |
| AtCoder | `atcoder.jp/contests/abc123/tasks/abc123_d` | `abc123d` | 去掉下划线 |
| AT hard version | `.../tasks/abc123_d1` | `abc123d1` | 去掉下划线 |

## 日常使用

```
WPS 编辑 mappings.xlsx → 保存关闭 → git commit → git push
                                              ↓
                              GitHub Actions 自动运行（约 30 秒）
                                              ↓
                         油猴脚本下次打开 OJ 页面时自动获取最新数据（1 小时缓存）
```

第一次 push 后，打开 `https://conanforever22.github.io/oj-resetta/data.json` 确认数据可访问。

## 页面效果

| 浏览页面 | 检测到 | 标签标题 / TM 菜单 |
|---------|-------|---------|
| luogu.com.cn/problem/P1001 | 已搬运 | 🟢 已搬运 → Hydro #100 |
| codeforces.com/problemset/problem/2038/A | 已搬运 | 🟢 已搬运 → Hydro #200 |
| atcoder.jp/contests/abc123/tasks/abc123_d | 已搬运 | 🟢 已搬运 → Hydro #300 |
| luogu.com.cn/problem/P9999 | 未搬运 | 无显示 |
| xxx.com/d/repo/p/100 | 来源 luogu | 🟢 来源: 洛谷 P1001 |
| xxx.com/d/repo/p/999 | 无来源 | 🟡 原创 |

## 新增 OJ

需要修改 3 处：

### 1. `mappings.xlsx`
新增工作表，填写映射。

### 2. `build.py`
在 `SHEET_OJ_MAP` 添加：
```python
SHEET_OJ_MAP = {
    ...
    "新OJ名":  "new_oj_key",    # ← 新增
}
```
如需自定义源 URL 生成逻辑，在 `SOURCE_URL_BUILDERS` 注册函数。

### 3. `oj-tracker.user.js`
- `@match` 添加新 OJ 的 URL 模式
- `SOURCE_DISPLAY_NAMES` 添加显示名称
- `detectPage()` 添加 URL 匹配和标准化逻辑
- `SOURCE_URL_TEMPLATES` 添加 URL 构造函数

## 隐私说明

- `data.json` 公开托管在 GitHub Pages，只包含 OJ 题号到自有题号的映射关系（不含任何域名、密码或敏感信息）
- Hydro 域名仅存在于本地油猴脚本中，不入库
- 如需迁移到私有部署，修改 `DATA_URL` 即可
