# Selenium 关键字驱动自动化测试框架

基于 Python + Selenium 的关键字驱动测试框架，通过 Excel 编写测试用例，配合反射机制实现无代码自动化测试。

## 目录结构

```
selenium-keywords/
├── Common/               # 核心框架
│   ├── KeyWords.py       # Selenium 关键字方法封装
│   ├── ref_invoke.py     # 反射调用引擎
│   ├── HandleTestCase.py # Excel 测试用例读取与执行
│   └── conf_dirs.py      # 目录路径配置
├── Utils/                # 工具模块
│   ├── HandleExcel.py    # Excel 读写封装
│   └── find_element_by_locator.py  # 元素定位器
├── Libs/                  # 第三方报告库
├── TestCases/            # unittest 测试用例 (ddt 数据驱动)
├── TestCases_py/         # pytest 测试用例
├── TestDatas/            # Excel 测试用例文件
├── HTMLReports/          # HTML 测试报告输出
├── Screenshots/          # 失败截图
├── Logs/                 # 日志
└── run_main.py           # unittest 入口
```

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

依赖：`selenium==3.141.0`、`xlrd==2.0.1`、`xlutils==2.0.0`、`xlwt==1.3.0`、`ddt==1.4.2`、`pytest==6.2.3`

### 2. 准备测试用例 Excel

在 `TestDatas/` 目录下放置 `关键字驱动测试用例.xlsx`，包含两个关键 Sheet：

- **测试用例** — 列出所有待执行的测试场景（Sheet 名）
- **[Sheet 名]** — 该 Sheet 下的步骤行，列定义如下：

| 步骤名称 | 关键字 | 定位方式 | 内容 |
|---------|--------|---------|------|
| 打开浏览器 | `open_browser` | | chrome |
| 打开网址 | `get_url` | | https://www.baidu.com |

### 3. 编写测试用例

每个步骤行的关键字列填写 `KeyWords.py` 中的方法名，定位方式和内容作为参数传入。

关键字方法：

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `open_browser` | browser (chrome/ie/ff) | 启动浏览器 |
| `get_url` | url | 打开网址 |
| `click_element` | locator | 点击元素 |
| `element_send_keys` | locator, content | 输入文本 |
| `assert_text` | locator, text | 断言页面包含文本 |
| `switch_frame` | locator | 切换 iframe |
| `save_screenshots_png` | | 截图 |
| `sleep_time` | | 强制等待 |
| `quit_browser` | | 关闭浏览器 |

定位方式格式：`By=value`，例如 `id=kw`、`xpath=//input[@id='kw']`

### 4. 运行测试

**unittest 模式（生成 HTML 报告）：**
```bash
python run_main.py
```

**pytest 模式：**
```bash
pytest                                    # 运行 TestCases 目录
pytest TestCases_py/test_baidu_by_pytest.py  # 运行指定文件
```

## 工作原理

```
Excel 测试用例
    ↓
HandleTestCase 读取 sheet 行
    ↓
(step_name, key_words, locator, content)
    ↓
ref_invoke.run_keywords_method()
    ↓
getattr(KeyWordsMethod, key_words)  ← Python 反射
    ↓
调用 Selenium 关键字方法
    ↓
执行结果回写 Excel
```

## 执行流程

1. `run_main.py` 启动 unittest，发现 `TestCases/test*.py`
2. `@ddt` 装饰器从 Excel 读取所有待执行 Sheet 名称
3. 每条用例调用 `HandleTestCase` 读取对应 Sheet 的步骤行
4. 每一步通过反射调用 `KeyWords.py` 中的同名方法
5. 执行结果（pass/failed）和耗时回写到 Excel 中
6. unittest 使用 `HTMLTestRunnerNew` 输出 HTML 报告到 `HTMLReports/`

## 注意事项

- Excel 读写使用不同库：**读取用 xlrd**（支持 .xlsx），**写入用 xlwt**（只支持 .xls）。如需写入结果，需准备双份文件或转换格式。
- `selenium==3.141.0` 较旧，浏览器驱动需单独安装并配置到 PATH。
- pytest.ini 将 `testpaths` 限制为 `./TestCases`，`TestCases_py/` 不受 pytest 自动发现，需手动指定路径运行。