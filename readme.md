# 校园失物招领平台

这是一个基于 Flask 框架开发的校园失物招领平台，旨在帮助学生和教职工发布、查找和管理校园内的失物与拾物信息。

## 功能概览

  * **用户管理**：注册、登录功能。
  * **信息发布**：发布失物或拾物信息，支持上传图片和在地图上标注具体位置。
  * **信息浏览**：查看所有已发布的失物/拾物列表和详细信息。
  * **个人中心**：用户可以管理自己发布的信息，包括编辑、删除和标记为已解决。
  * **站内消息**：便捷的站内私信功能，方便失主与拾物者直接沟通。
  * **邮件通知**：支持通过邮件发送通知（例如，用户注册成功、密码重置或收到站内消息）。
-----

## 快速上手

### 1\. 环境准备

在开始前，请确保你的系统已安装 **Python 3.8+**、**pip** 和 **MySQL 服务器**。

### 2\. 设置虚拟环境

强烈建议使用虚拟环境来管理项目依赖。

```bash
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# macOS/Linux:
source venv/bin/activate
# Windows (Command Prompt):
venv\Scripts\activate.bat
# Windows (PowerShell):
venv\Scripts\Activate.ps1
```

### 3\. 安装依赖

激活虚拟环境后，安装项目所需的所有 Python 库。

```bash
pip install -r requirements.txt
```

-----

### 4\. 数据库配置 (MySQL)

本项目使用 MySQL 数据库。请按照以下步骤进行设置：

1.  **启动你的 MySQL 服务器。**

2.  **创建数据库和表：**
    连接到你的 MySQL 服务器（例如，通过命令行输入 `mysql -u root -p` 并输入密码），然后执行以下 SQL 命令来创建 `lost_and_found` 数据库和所需的表结构：

    ```sql
    -- 创建数据库
    CREATE DATABASE IF NOT EXISTS lost_and_found CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

    -- 连接到数据库
    USE lost_and_found;

    -- 创建 users 表
    CREATE TABLE `users` (
      `id` int NOT NULL AUTO_INCREMENT,
      `username` varchar(80) COLLATE utf8mb4_unicode_ci NOT NULL,
      `email` varchar(120) COLLATE utf8mb4_unicode_ci NOT NULL,
      `password_hash` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `username` (`username`),
      UNIQUE KEY `email` (`email`)
    ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- 创建 items 表
    CREATE TABLE `items` (
      `id` int NOT NULL AUTO_INCREMENT,
      `type` varchar(10) COLLATE utf8mb4_unicode_ci NOT NULL,
      `name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL,
      `description` text COLLATE utf8mb4_unicode_ci,
      `location` varchar(200) COLLATE utf8mb4_unicode_ci NOT NULL,
      `event_time` datetime NOT NULL,
      `image_file` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT 'default.jpg',
      `user_id` int NOT NULL,
      `posted_date` datetime NOT NULL,
      `status` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'active',
      `pin_x` int DEFAULT NULL,
      `pin_y` int DEFAULT NULL,
      PRIMARY KEY (`id`),
      KEY `idx_items_user_id` (`user_id`),
      CONSTRAINT `fk_items_users` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`),
      CONSTRAINT `items_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=35 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- 创建 messages 表
    CREATE TABLE `messages` (
      `id` int NOT NULL AUTO_INCREMENT,
      `sender_id` int NOT NULL,
      `receiver_id` int NOT NULL,
      `item_id` int DEFAULT NULL,
      `content` text COLLATE utf8mb4_unicode_ci NOT NULL,
      `timestamp` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
      `is_read` tinyint(1) NOT NULL DEFAULT '0',
      PRIMARY KEY (`id`),
      KEY `sender_id` (`sender_id`),
      KEY `receiver_id` (`receiver_id`),
      KEY `item_id` (`item_id`),
      CONSTRAINT `messages_ibfk_1` FOREIGN KEY (`sender_id`) REFERENCES `users` (`id`),
      CONSTRAINT `messages_ibfk_2` FOREIGN KEY (`receiver_id`) REFERENCES `users` (`id`),
      CONSTRAINT `messages_ibfk_3` FOREIGN KEY (`item_id`) REFERENCES `items` (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
    ```

3.  **配置 `config.py`：**
    打开项目根目录下的 `config.py` 文件。
    将 `SQLALCHEMY_DATABASE_URI` 中的 `root:password@localhost` 替换为你的 MySQL 数据库的**实际用户名和密码**（例如 `your_mysql_username:your_mysql_password@your_mysql_host`）。同时，你也可以考虑将 `SECRET_KEY` 替换为一个固定且更安全的字符串。

    ```python
    # config.py
    import os

    class Config:
        # SECRET_KEY 用于 Flask 会话、CSRF 保护等。
        # 生产环境建议从环境变量获取，例如 os.environ.get('SECRET_KEY')
        SECRET_KEY = os.urandom(24).hex() # <-- **请修改为一个固定且安全的字符串！**

        # MySQL 数据库连接 URI： 'mysql+pymysql://用户名:密码@主机/数据库名'
        SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://root:password@localhost/lost_and_found' # <-- **请替换为你的实际数据库凭据！**
        SQLALCHEMY_TRACK_MODIFICATIONS = False

        # 图片上传目录，确保 static/uploads 目录存在且有写入权限
        UPLOAD_FOLDER = os.path.join(os.path.abspath(os.path.dirname(__file__)), 'static/uploads')
        MAX_CONTENT_LENGTH = 16 * 1024 * 1024 # 最大文件上传大小 16 MB
    ```


4.  **邮件服务器配置：**
    务必将以下占位符替换为你实际的邮件服务提供商信息。 例如，如果你使用 Gmail，可能需要开启“两步验证”并生成“应用专用密码”。如果你使用 QQ 邮箱，则需要使用“授权码”。
    ```python
    # apps.py
    MAIL_USERNAME = 'your_email@example.com' # 你的发件邮箱地址
    MAIL_PASSWORD = 'your_email_app_password_or_auth_code'  # 你的邮箱应用专用密码或授权码
    MAIL_DEFAULT_SENDER = 'your_email@example.com' # 默认发件人
    ```
-----

### 5\. 运行应用程序

在项目根目录下，确保虚拟环境已激活，然后执行：

```bash
python app.py
```

应用程序将会在 `http://127.0.0.1:5000/` 运行。在浏览器中打开此地址即可开始使用。

-----


## 备注

  * **项目结构**：项目文件应组织如下：`app.py`（主应用文件）、`models.py`（数据库模型）、`config.py`（配置文件）、`requirements.txt`（依赖列表）、`static/`（包含 `css/`, `images/`, `map/`, `uploads/` 等）、`templates/`（HTML 模板）。
  * **生产部署**：对于生产环境，不建议直接使用 `python app.py`。请使用 Gunicorn (Linux) 或 Waitress (Windows) 等生产级 WSGI 服务器来运行你的 Flask 应用。
  * **安全性**：在生产环境中，**`SECRET_KEY` 和数据库凭据应通过环境变量管理**，切勿硬编码。
  * **图片上传目录**：请确保 `static/uploads` 目录存在且具有写入权限。

-----


## 项目结构 (供参考)

```
your_project_name/
├── app.py              # Flask 应用主文件，包含应用初始化和主要路由
├── models.py           # 数据库模型定义 (SQLAlchemy)
├── config.py           # 应用程序配置
├── requirements.txt    # 项目依赖列表
├── static/
│   ├── css/            # CSS 样式文件
│   │   └── style.css
│   ├── js/             # JavaScript 文件 (如果有)
│   ├── images/         # 网站图标、图钉等图片
│   │   └── pin.png
│   ├── map/            # 校园地图图片
│   │   └── map.png
│   └── uploads/        # 用户上传的物品图片 (此目录需要可写权限)
├── templates/
│   ├── index.html
│   ├── login.html
│   ├── register.html
│   ├── publish_item.html
│   ├── item_detail.html
│   ├── dashboard.html
│   ├── edit_item.html
│   ├── my_messages.html
│   └── ... (其他 HTML 模板文件)
└── emails/             # 邮件模板文件
    ├── email_match_notification.html
    └── email_new_message.html
└── README.md           # 本文件
```

-----
## 权利保留

本项目采用 **MIT 许可证**。你可以自由使用、修改和分发本代码，但需保留原始版权声明。详情请参见项目根目录下的 `LICENSE` 文件或以下许可证文本：

```
MIT License

Copyright (c) [年份] [你的名字或组织名称]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

-----

[//]: # (## 引用方式 &#40;Citing&#41;)

[//]: # ()
[//]: # (如果你在学术工作或公共项目中使用了本平台代码，请考虑引用本项目。以下是一个建议的引用格式（请根据实际情况填写年份和作者信息）：)

[//]: # ()
[//]: # (**通用格式：**)

[//]: # ()
[//]: # (```)

[//]: # ([作者姓名/组织名称]. &#40;年份&#41;. 校园失物招领平台. [项目URL或代码仓库地址].)

[//]: # (```)

[//]: # ()
[//]: # (**BibTeX 格式 &#40;适用于 LaTeX 文档&#41;：**)

[//]: # ()
[//]: # (```bibtex)

[//]: # (@misc{lostfound_platform_[年份],)

[//]: # (  author = {[作者姓氏, 名] 或 {您的组织名称}},)

[//]: # (  title = {{校园失物招领平台}},)

[//]: # (  year = {[年份]},)

[//]: # (  howpublished = {\url{[你的项目Git仓库地址]}},)

[//]: # (  note = {Accessed: [访问日期, 例如: 2025年6月30日]})

[//]: # (})

[//]: # (```)

[//]: # ()
[//]: # (**示例：**)

[//]: # ()
[//]: # (```)

[//]: # (@misc{lostfound_platform_2025,)

[//]: # (  author = {Zhang, San},)

[//]: # (  title = {{校园失物招领平台}},)

[//]: # (  year = {2025},)

[//]: # (  howpublished = {\url{https://github.com/your-username/your-repo-name}},)

[//]: # (  note = {Accessed: 2025年6月30日})

[//]: # (})

[//]: # (```)
