# AGENTS.md

## Project Overview
Python Selenium keyword-driven web automation framework. Uses Python reflection to call keyword methods by name from Excel test cases. No package manager (plain Python project).

## Directory Structure
- `Common/` - Core framework: `KeyWords.py` (selenium wrappers), `ref_invoke.py` (reflection executor), `conf_dirs.py` (path config), `HandleTestCase.py` (Excel-based case execution)
- `Utils/` - Utilities: `HandleExcel.py`, `find_element_by_locator.py`, `HandleLogging.py`, `DateTimeFormat.py`
- `Libs/` - Third-party test libs: HTMLTestRunner (unittest), HTMLTestReportCN
- `TestCases/` - unittest-based test cases (`test_baidu_by_unittest.py`)
- `TestCases_py/` - pytest-based test cases (`test_baidu_by_pytest.py`)
- `TestDatas/` - Excel test case files (`关键字驱动测试用例.xlsx`)
- `HTMLReports/` - Generated HTML reports
- `Scripts/` - Legacy AutoTest*.py versions

## Key Commands
- Run all unittest cases: `python run_main.py`
- Run pytest cases: `pytest` (reads `pytest.ini` which sets `testpaths = ./TestCases`)
- Run a specific test: `pytest TestCases_py/test_baidu_by_pytest.py`
- Install deps: `pip install -r requirements.txt`

## Important Conventions
- `pytest.ini` restricts pytest to `./TestCases` only — `TestCases_py/` is for manual pytest runs
- `Common/ref_invoke.py` uses Python reflection (`getattr`) to call methods on `KeyWordsMethod` by keyword string name
- Keyword method names in Excel must exactly match method names in `KeyWords.py` (e.g., `open_browser`, `get_url`, `click_element`)
- `HandleExcel` uses xlrd/xlutils/xlwt — file format must be `.xls` for writing, `.xlsx` for reading
- `run_main.py` loads `excel_path + "/关键字驱动测试用例.xlsx"` as its test data file
- `TestCases/test_baidu_by_unittest.py` uses `@ddt` decorator from `ddt` library
- `selenium==3.141.0` — older version, browser drivers must be installed separately

## Architecture Notes
- Keyword execution flow: Excel row → (step_name, key_words, locator, content) → `ref_invoke.run_keywords_method()` → `getattr(KeyWordsMethod, key_words)` → call method with args
- `KeyWords.py` maintains `self.driver` as instance state; all keyword methods use it
- Screenshots saved to `Screenshots/`, logs to `Logs/` (paths via `conf_dirs.py`)
- `AutoTest3.py` is a standalone tkinter GUI app, not integrated with the test runner
