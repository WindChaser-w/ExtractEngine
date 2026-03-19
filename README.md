ExtractEngine 自动化测试框架

一个基于 Python + Pytest 的接口自动化测试框架，采用关键字驱动设计，支持 YAML 编写用例

 ✨ 特性

- 📝 **YAML 驱动测试**：测试人员只需编写 YAML 文件，无需编写 Python 代码
- 🔄 **接口依赖处理**：通过 `extract.yaml` 全局存储中间变量，实现跨用例参数传递
- 🎯 **动态参数解析**：支持 `${}` 表达式，灵活调用时间戳、随机数等自定义函数
- 🔍 **双引擎数据提取**：同时支持 JSONPath 和正则表达式提取响应数据
- ✅ **多类型断言**：支持相等、包含、数据库等多种断言方式
- 📊 **Allure 报告**：生成美观的可视化测试报告，支持历史趋势
- 🔧 **CI/CD 集成**：无缝对接 Jenkins，实现自动化执行与通知

---

## 🛠️ 技术栈

Python 3.8+ + Pytest + Requests + PyYAML + Allure + JSONPath + SQLAlchemy + Jenkins + Loguru

---

## 📁 项目结构

```
ExtractEngine/
├── base/               # 核心引擎
│   ├── apiutil.py      # 请求封装与用例执行
│   └── generateId.py   # ID生成器
├── common/             # 工具类
│   ├── assertions.py   # 断言封装
│   ├── debugtalk.py    # 自定义函数库
│   ├── readyaml.py     # YAML读写
│   └── recordlog.py    # 日志记录
├── conf/               # 配置文件
│   ├── config.ini      # 环境配置
│   └── setting.py      # 全局设置
├── data/               # 测试数据
├── logs/               # 运行日志
├── report/             # 测试报告
├── testcase/           # YAML测试用例
├── .gitignore          # Git忽略文件
├── pytest.ini          # Pytest配置
├── requirements.txt    # 依赖清单
└── run.py              # 启动入口
```

---

## 🚀 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 配置环境

编辑 `conf/config.ini`，设置接口地址：

```ini
[api_envi]
host = http://your-api-server.com
```

### 3. 编写测试用例

在 `testcase/` 目录下创建 YAML 文件，例如 `test_login.yaml`：

```yaml
baseInfo:
  api_name: 登录接口
  url: /user/login
  method: post
  header:
    Content-Type: application/json
testCase:
  - case_name: 正确密码登录
    data:
      username: admin
      password: 123456
    validation:
      - eq: {code: 200}
    extract:
      token: $.data.token
```

### 4. 运行测试

```bash
python run.py
```

---

## 📖 核心功能详解

### 动态参数 `${}`

在 YAML 中使用 `${}` 调用 `common/debugtalk.py` 中的自定义函数：

```yaml
data:
  timestamp: ${current_time()}
  phone: ${random_phone()}
  token: ${get_extract_data(token)}
```

### 接口依赖处理

登录用例提取 token 存入 `extract.yaml`，后续用例自动引用：

```yaml
# 登录用例
extract:
  token: $.data.token

# 下单用例
headers:
  Authorization: Bearer ${get_extract_data(token)}
```

### 数据提取

支持 JSONPath 和正则表达式：

```yaml
extract:
  token: $.data.token                    # JSONPath
  user_id: '"userId":(\d+)'               # 正则
  order_ids: $.data.orders[*].id          # 列表提取
```

### 断言验证

支持多种断言方式：

```yaml
validation:
  - eq: {code: 200}                       # 相等断言
  - contains: {message: "success"}         # 包含断言
  - db: "select * from user where id=1"    # 数据库断言
```

---

## 🔧 自定义函数库

在 `common/debugtalk.py` 中添加自定义函数，YAML 中可直接调用：

```python
class DebugTalk:
    def current_time(self):
        import time
        return time.strftime('%Y-%m-%d %H:%M:%S')
    
    def random_phone(self):
        import random
        return f"139{random.randint(10000000, 99999999)}"
```

---

## 📊 测试报告

运行后生成 Allure 报告：

```bash
# 自动打开浏览器查看报告
python run.py
```

报告包含：
- 测试概览与统计
- 用例执行详情
- 请求/响应数据
- 环境信息
- 历史趋势

---

## 🔄 CI/CD 集成

### Jenkins 配置示例

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { git 'https://github.com/WindChaser-w/ExtractEngine.git' }
        }
        stage('Install Dependencies') {
            steps { sh 'pip install -r requirements.txt' }
        }
        stage('Run Tests') {
            steps { sh 'python run.py' }
        }
        stage('Allure Report') {
            steps { allure includeProperties: false, results: [[path: 'report/temp']] }
        }
    }
}
```

---

## ⚙️ 配置说明

### pytest.ini

```ini
[pytest]
filterwarnings =
    error
    ignore::UserWarning
python_files = test_*.py
python_classes = Test*
python_functions = test
```

### setting.py

```python
REPORT_TYPE = 'allure'  # allure 或 tm
LOG_LEVEL = 'INFO'
FILE_PATH = {
    'EXTRACT': 'extract.yaml',
    'YAML': './testcase'
}
```

---

## 🤝 参与贡献

欢迎提交 Issue 或 Pull Request！

1. Fork 本仓库
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的修改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

---

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

## 📧 联系作者

- GitHub：[@WindChaser-w](https://github.com/WindChaser-w)
- 邮箱：wpy2306@outlook.com
- 博客：https://blog.csdn.net/black_snack/article/details/158810936?fromshare=blogdetail&sharetype=blogdetail&sharerId=158810936&sharerefer=PC&sharesource=black_snack&sharefrom=from_link

---

**如果这个框架对你有帮助，欢迎 Star ⭐ 支持！**
