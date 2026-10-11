# 接口 + UI 双端自动化测试（pytest + Playwright）

![接口测试](https://github.com/childish999/api-ui-autotest-portfolio/actions/workflows/api-tests.yml/badge.svg)

基于 pytest 的自动化测试项目。接口端测试 [JSONPlaceholder](https://jsonplaceholder.typicode.com) 的公开接口，UI 端测试必应搜索，共 17 条用例。运行一次 pytest 会生成 HTML 报告；失败用例的截图、录像和 Trace 文件会自动保留。

## 技术栈

- Python 3.12
- pytest：用例组织、fixture、参数化
- requests：接口请求
- pytest-playwright（Playwright）：浏览器自动化
- pytest-html：HTML 测试报告
- pytest-rerunfailures：失败自动重跑

## 用例覆盖

接口端（JSONPlaceholder，15 条）：

- GET 查询（11 条）：正常查询，以及 id 为 0、负数、非数字、超大值、不存在的参数化异常流
- POST 创建（2 条）：正常创建；空对象基线，确认接口不为缺失字段补默认值
- DELETE 与生命周期（2 条）：POST、DELETE 参数关联，测试数据由 fixture 自动清理

UI 端（必应搜索，2 条）：

- 中文搜索“接口测试”，英文搜索“Playwright”，运行在 chromium 上
- 页面操作封装在 BingPage 页面对象中，用例只写流程和断言

## 工程化亮点

- 夹具统一写在 conftest.py，夹具之间可以链式取值。用例结束后自动 DELETE 造出来的帖子，用 yield 做后置清理，用例失败也不会漏删
- 网络抖动造成的偶发失败会自动重跑两次（--reruns 2），减少误报。实际跑套件时遇到过偶发超时，第一遍失败、重跑后通过
- 失败用例会保留截图、录像和 trace.zip（截图用 only-on-failure，录像和 tracing 用 retain-on-failure），用 playwright show-trace 可以回放整个过程；通过的用例不生成这些文件
- 定位器只在 tests/pages/ 下存一份，页面操作和测试逻辑分开。页面改版时只改页面对象，用例不动

## 项目结构

```
TestPythonProject2/
├── hello_browser.py             # C-1 的第一个脚本，留作纪念
├── conftest.py                  # 全局 fixture：base_url、造帖与清理、bing_page
├── pytest.ini                   # 报告、重跑、失败证据的开关
├── requirements.txt             # 依赖清单
├── .gitignore                   # 不收录报告、缓存、失败证据文件
├── report.html                  # 测试报告（运行后生成）
└── tests/
    ├── test_read_posts.py       # 接口：查询（含参数化异常流）
    ├── test_create_post.py      # 接口：创建
    ├── test_delete_lifecycle.py # 接口：删除与生命周期
    ├── test_ui_search.py        # UI：必应搜索
    └── pages/
        └── bing_page.py         # 必应页面对象
```

## 运行方式

```bash
pip install -r requirements.txt
playwright install chromium      # UI 端首次运行前需安装浏览器内核
pytest                           # 运行全部 17 条用例
pytest tests/test_read_posts.py  # 只跑接口端
pytest tests/test_ui_search.py   # 只跑 UI 端
```

运行结束后打开 report.html 查看结果，失败用例的证据文件在 test-results/ 目录下。
