## 前言

最近突然想起年前图床仓库发生的一个遗留问题：由于我的网络图床服务是`Github + Typora`的形式，本地的图片会自动转义成网络图片并存储在图床仓库下，一般我们会指定一个目录进行图片存储，但是由于GitHub设定的单个目录最大存储文件数不能超过1000.

所以在注意到这件事的情况下GitHub的图床仓库就发生了问题：新加入的图片文件由于没有文件位，会自动代替旧的图片文件，这就导致了部分文件的丢失，所以这里想写一个GitHub仓库的自动化Action，每天检测仓库下每个目录下的文件个数，超过999个文件自动给GitHub默认绑定的邮箱发送信息提醒。

## 具体流程

当每天自动检测仓库中每个目录中的文件数量，并且如果超过999个文件时，自动向与GitHub账户关联的默认邮箱发送消息。

### 1. 创建GitHub工作流文件
在GitHub仓库中，转到`.github/workflows`目录并创建一个新文件，比如`file_count.yml`。该文件将定义运行自动化操作的工作流。

### 2. 定义工作流
在`file_count.yml`文件中，添加以下代码：

```yaml
name: File Count Reminder

on:
  schedule:
    - cron: "0 0 * * *" # Runs every day at midnight UTC

jobs:
  count-files:
    runs-on: ubuntu-latest

    steps:
      - name: Check out code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.10' # Replace with the desired Python version

      - name: Count files and send email
        run: |
          pip install -r requirements.txt
          python send_email.py ${{ secrets.GITHUB_TOKEN }}
```

### 3. 创建`requirements.txt`文件
在GitHub仓库中创建一个名为`requirements.txt`的文件，并将以下内容添加到文件中：

```
py-emails
```

### 4. 创建`send_email.py`文件
在GitHub仓库中创建一个名为`send_email.py`的文件，并将以下代码添加到文件中：

```python
import os
import smtplib
from email.mime.text import MIMEText
from email.header import Header

def count_files(directory):
    file_count = 0
    for root, dirs, files in os.walk(directory):
        file_count += len(files)
    return file_count

def send_email(github_token, recipient, file_count):
    smtp_server = 'smtp.gmail.com'
    smtp_port = 587

    subject = 'File Count Reminder'
    content = f'The repository has {file_count} files.'

    message = MIMEText(content, 'plain', 'utf-8')
    message['From'] = Header('GitHub Action')
    message['To'] = Header(recipient)
    message['Subject'] = Header(subject)

    try:
        server = smtplib.SMTP(smtp_server, smtp_port)
        server.starttls()
        server.login('githubaction@gmail.com', github_token)
        server.sendmail('githubaction@gmail.com', recipient, message.as_string())
        server.quit()
        print("Email reminder sent to", recipient)
    except Exception as e:
        print("Failed to send email:", str(e))

repository_path = '.'  # Replace with the path to your repository if needed
file_limit = 999

file_count = count_files(repository_path)
if file_count > file_limit:
    github_token = os.environ.get('INPUT_GITHUB_TOKEN')
    default_email = os.environ.get('GITHUB_ACTOR') + '@users.noreply.github.com'

    send_email(github_token, default_email, file_count)
else:
    print("The repository has", file_count, "files. No reminder needed.")
```

使用这些步骤，工作流将每天UTC时间午夜运行，计算仓库中的文件数量，如果文件数量超过999，则会向与GitHub账户关联的默认邮箱发送邮件提醒。