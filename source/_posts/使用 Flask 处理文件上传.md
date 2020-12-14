---
title: 使用 Flask 处理文件上传
date: 2020-04-12
updated: 2020-04-12
tags: [Flask,Python]
categories: [Flask]
---

# 使用 Flask 处理文件上传

Web 应用程序的一个常见特性是允许用户将文件上传到服务器。在 [RFC 1867](https://tools.ietf.org/html/rfc1867) 中协议记录了客户端上传文件的机制，我们最喜欢的 Web 框架 Flask 完全支持这一机制，但是对于许多开发者来说，还有许多实现细节未遵循该正式规范。诸如在何处存储上传的文件，如何事后使用它们，或者如何保护服务器不受恶意文件上传的影响，这些都会产生很多混乱和不确定性。

<!-- more -->

在本文中，我将向你展示如何为 Flask 服务器实现强大的文件上传功能，该功能不仅支持基于 Web 浏览器中的标准文件上传并且与基于 JavaScript 的上传小部件兼容：


![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1602214447739-a6c5a7a3-75ec-4082-87e7-efb8db03f878.gif)

## 基本文件上传表单
从高层次的角度来看，上传文件的客户端与其他任何表单数据提交一样。 换句话说，你必须定义一个包含文件字段的 HTML 表单。

下面是一个简单的 HTML 页面，该表单接受一个文件:
```html
<!doctype html>
<html>
  <head>
    <title>File Upload</title>
  </head>
  <body>
    <h1>File Upload</h1>
    <form method="POST" action="" enctype="multipart/form-data">
      <p><input type="file" name="file"></p>
      <p><input type="submit" value="Submit"></p>
    </form>
  </body>
</html>
```
![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1602214835402-35423468-f037-42a5-95b0-10a06591fc04.png)
你可能知道，`<form>` 元素的 method 属性可以是 `GET` 或 `POST`。使用 `GET` 时，数据将在请求 URL 的查询字符串中提交，而使用 `POST` 时，数据将进入请求主体。在表单中包含文件时，必须使用 `POST`，因为不可能在查询字符串中提交文件数据。

没有文件的表单通常不包含 `<form>` 元素中的 `enctype` 属性。此属性定义浏览器在将数据提交到服务器之前应该如何格式化数据。HTML 规范为其定义了三个可能的值：

- `application/x-www-form-urlencoded`: 

这是默认格式，也是不包含文件字段的表单的最佳格式

- `multipart/form-data`: 

如果表单中至少有一个字段是文件字段，则需要此格式

- `text/plain`: 

这种格式没有实际用途，所以你应该忽略它

实际的文件字段是我们用于大多数其他表单字段的标准 `<input>` 元素，其类型设置为 `file`。 在上面的示例中，我没有包含任何其他属性，但是file字段支持两个有时有用的属性：


- `multiple`: 

可用于允许在单个文件字段中上载多个文件。例如:
```html
<input type="file" name="file" multiple>
```


- `accept`: 

可以用于筛选允许的文件类型，这些文件类型可以通过文件扩展名或媒体类型选择。例子:
```html
<input type="file" name="doc_file" accept=".doc,.docx">
<input type="file" name="image_file" accept="image/*">
```
## 
## 使用 Flask 接受文件提交
对于常规表单，Flask 提供了对 `request.form` 字典中提交的表单字段的访问。 但是，文件字段包含在`request.files` 字典中。 `request.form` 和 `request.files` 字典实际上是“multi-dicts”，它是一种支持重复键的专门字典实现。 这是必要的，因为表单可以包含多个具有相同名称的字段，通常情况下是由多组复选框组成。 对于允许多个文件的文件字段，也会发生这种情况。


暂时忽略诸如验证和安全性等重要方面，下面简短的 Flask 应用程序接受使用上一节中定义的表单上传的文件，并将提交的文件写入当前目录：
```python
from flask import Flask, render_template, request, redirect, url_for
app = Flask(__name__)
@app.route('/')
def index():
    return render_template('index.html')
@app.route('/', methods=['POST'])
def upload_file():
    uploaded_file = request.files['file']
    if uploaded_file.filename != '':
        uploaded_file.save(uploaded_file.filename)
    return redirect(url_for('index'))
```


upload_file() 函数使用@app.route装饰，以便在浏览器发送POST请求时调用该函数。 请注意，同一个根 URL 是如何在两个视图函数之间进行拆分的，并将 `index()` 设置为接受 `GET` 请求，将 `upload_file``()` 上传为 `POST` 请求。

`uploaded_file` 变量保存提交的文件对象。 这是 Flask 从 Werkzeug 导入的 `FileStorage` 类的实例。


`FileStorage` 中的 `filename` 属性提供客户端提交的文件名。如果用户提交表单时没有在 file 字段中选择文件，那么文件名将是一个空字符串，因此始终检查文件名以确定文件是否可用是很重要的。


Flask 收到文件提交后，不会自动将其写入磁盘。 这实际上是一件好事，因为它使应用程序有机会查看和验证文件提交，这一点将在后面看到。 可以从 stream 属性访问实际文件数据。 如果应用程序只想将文件保存到磁盘，则可以调用 `save()` 方法，并将所需路径作为参数传递。 如果未调用文件的 `save()` 方法，则该文件将被丢弃。

是否要使用此应用程序测试文件上传？ 为你的应用程序创建目录，并将上面的代码编写为 app.py。 然后创建一个模板子目录，并将上一节中的HTML页面编写为templates/index.html。 创建一个虚拟环境并在其上安装Flask，然后使用 `flask run` 运行该应用程序。 每次提交文件时，服务器都会把它的副本写到当前目录中。

在继续讨论安全性主题之前，我将讨论上面的代码的一些变体，你可能会发现这些变体很有用。 如前所述，可以将文件上传字段配置为接受多个文件。 如果像上面那样使用 `request.files['file']`，则只会得到一个提交的文件，但是使用 `getlist()` 方法，你可以在for循环中访问所有文件：
```python
for uploaded_file in request.files.getlist('file'):
        if uploaded_file.filename != '':
            uploaded_file.save(uploaded_file.filename)
```


许多人在 Flask 中编写表单处理路由时，对 GET 和 POST 请求使用单个视图函数。使用单视图函数的示例应用程序的版本编码如下:
```python
@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        uploaded_file = request.files['file']
        if uploaded_file.filename != '':
            uploaded_file.save(uploaded_file.filename)
        return redirect(url_for('index'))
    return render_template('index.html')
```


最后，如果使用 [Flask-WTF](https://flask-wtf.readthedocs.io/en/stable/) 扩展来处理表单，则可以使用 `FileField` 对象上传文件。到目前为止，你看到的例子中使用的表单可以使用 Flask-WTF 编写如下:
```python
from flask_wtf import FlaskForm
from flask_wtf.file import FileField
from wtforms import SubmitField
class MyForm(FlaskForm):
    file = FileField('File')
    submit = SubmitField('Submit')
```


注意，`FileField` 对象来自 `flask_wtf` 包，与大多数其他字段类不同，后者直接从 `wtforms` 包导入。Flask-WTF 为文件字段提供了两个验证器，FileRequired 和 FileAllowed，前者执行类似于空字符串检查的检查，后者确保文件扩展名包含在允许的扩展名列表中。

当您使用 Flask-WTF 表单时，file 字段对象的 `data` 属性指向 FileStorage 实例，因此将文件保存到磁盘的工作方式与上面的示例相同。
## 保护文件上传
上一节中给出的文件上传示例是一个非常简单的实现，不是很健壮。Web 开发中最重要的规则之一是永远不要信任客户提交的数据，因此在使用常规表单时，像 Flask-WTF 这样的扩展会在接受表单和整合数据到应用程序中之前对所有字段进行严格验证。对于包含文件字段的表单，也需要进行验证，因为如果不进行文件验证，服务器将为攻击敞开大门。例如:

- 攻击者可以上传一个非常大的文件，以至于服务器中的磁盘空间完全被填满，从而导致服务器出现故障
- 攻击者可以使用文件名（例如../../../.bashrc或类似文件）的上传请求，以试图欺骗服务器重写系统配置文件。
- 攻击者可以上传带有病毒或其他类型恶意软件的文件到应用程序需要使用的位置，例如，用户头像
### 
### 限制上传文件的大小
为了防止客户端上传非常大的文件，您可以使用 Flask 提供的配置选项。`MAX_CONTENT_LENGTH`  选项控制请求主体可以拥有的最大大小。虽然这不是一个特定于文件上传的选项，但设置一个最大的请求体大小有效地使 Flask  使用413状态码丢弃大于允许的请求体大小的请求


让我们修改上一节中的 app.py 示例，只接受最大为1 MB 的请求:
```python
app.config['MAX_CONTENT_LENGTH'] = 1024 * 1024
```


如果你试图上传一个大于1 MB 的文件，应用程序现在将拒绝它。


### 验证文件名
我们不能完全相信客户端提供的文件名是有效的和可以安全使用的，所以随上传文件一起提供的文件名必须经过验证。

要执行的一个非常简单的验证是确保文件扩展名是应用程序愿意接受的扩展名，这与使用 Flask-WTF 时F `FileAllowed` 验证器所做的类似。假设应用程序接受图像，那么它可以配置允许的文件扩展名列表:
```python
app.config['UPLOAD_EXTENSIONS'] = ['.jpg', '.png', '.gif']
```


对于每个上传的文件，应用程序可以确保文件扩展名是允许的:
```python
filename = uploaded_file.filename
if filename != '':
    file_ext = os.path.splitext(filename)[1]
    if file_ext not in current_app.config['UPLOAD_EXTENSIONS']:
        abort(400)
```


使用这种逻辑，任何不在允许的文件扩展名的文件名，都会出现400错误。

除了文件扩展名之外，验证文件名以及提供的任何路径也很重要。 如果你的应用程序不关心客户端提供的文件名，则处理上传的最安全方法是忽略客户端提供的文件名，而是生成自己的文件名，然后传递给 `save()` 方法。 这种技术工作良好的示例是头像上传。 每个用户的头像都可以使用用户 ID 保存为文件名，因此客户端提供的文件名可以丢弃。 如果你的应用程序使用 Flask-Login，则可以实现以下 `save()` 调用：
```python
uploaded_file.save(os.path.join('static/avatars', current_user.get_id()))
```


在其他情况下，保留客户端提供的文件名可能更好，因此必须首先清理文件名。对于这些情况，Werkzeug 提供了 [secure_filename()](https://werkzeug.palletsprojects.com/en/1.0.x/utils/#werkzeug.utils.secure_filename) 函数。让我们通过在 Python shell 中运行一些测试来看看这个函数是如何工作的:
```python
>>> from werkzeug.utils import secure_filename
>>> secure_filename('foo.jpg')
'foo.jpg'
>>> secure_filename('/some/path/foo.jpg')
'some_path_foo.jpg'
>>> secure_filename('../../../.bashrc')
'bashrc'
```


正如你在示例中看到的，无论文件名有多么复杂或多么恶意，secure_filename()  函数都将其缩减为一个单位文件名。

让我们将 `secure_filename()` 合并到示例上传服务器中，并添加一个配置变量，该变量定义文件上传的专用位置。下面是带有安全文件名的完整 app.py 源文件:
```python
import os
from flask import Flask, render_template, request, redirect, url_for, abort
from werkzeug.utils import secure_filename
app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 1024 * 1024
app.config['UPLOAD_EXTENSIONS'] = ['.jpg', '.png', '.gif']
app.config['UPLOAD_PATH'] = 'uploads'
@app.route('/')
def index():
    return render_template('index.html')
@app.route('/', methods=['POST'])
def upload_files():
    uploaded_file = request.files['file']
    filename = secure_filename(uploaded_file.filename)
    if filename != '':
        file_ext = os.path.splitext(filename)[1]
        if file_ext not in app.config['UPLOAD_EXTENSIONS']:
            abort(400)
        uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))
    return redirect(url_for('index'))
```
### 
> 注意
> secure_filename 函数将过滤所有非ASCII字符，因此，如果filename 是 "头像.jpg"之类的，则结果为"jpg"，但没有格式，这是个问题，我建议使用uuid模块重命名上传的文件，以避免出现上述情况。



### 验证文件内容
我将要讨论的第三层验证是最复杂的。如果您的应用程序接受某种文件类型的上传，那么理想情况下，它应该执行某种形式的内容验证，并拒绝任何不同类型的文件。

如何实现内容验证在很大程度上取决于应用程序接受的文件类型。对于本文中的示例应用程序，我使用的是图像，因此可以使用 Python 标准库中的  [imghdr](https://docs.python.org/3/library/imghdr.html) 包验证文件头实际上是一个图像。

让我们编写一个 `validate_image()` 函数，对图像执行内容验证:
```python
import imghdr
def validate_image(stream):
    header = stream.read(512)
    stream.seek(0)
    format = imghdr.what(None, header)
    if not format:
        return None
    return '.' + (format if format != 'jpeg' else 'jpg')
```


这个函数以一个字节流作为参数。它首先从流中读取512个字节，然后重置流指针，因为稍后当调用 save ()函数时，我们希望它看到整个流。前512字节的图像数据将足以识别图像的格式。

如果第一个参数是文件名，`imghdr.what()` 函数可以查看存储在磁盘上的文件; 如果第一个参数是 `None`，数据在第二个参数中传递，则可以查看存储在内存中的数据。`FileStorage` 对象为我们提供了一个流，因此最方便的选项是从它中读取安全数量的数据，并在第二个参数中将其作为字节序列传递。

imghdr.what() 的返回值是检测到的图像格式。该函数支持多种格式，其中包括流行的 `jpeg`、 `png` 和 `gif`。如果未检测到已知的图像格式，则返回值为 `None`。如果检测到格式，则返回该格式的名称。最方便的是将格式作为文件扩展名返回，因为应用程序可以确保检测到的扩展名与文件扩展名匹配，所以 validate_image() 函数将检测到的格式转换为文件扩展名。这很简单，只需为除 `jpeg` 外的所有图像格式添加一个点作为前缀，`jpeg` 除外，通常使用 .jpg扩展名。

下面是完整的 app.py，包含前面几节中的所有特性和内容验证:
```python
import imghdr
import os
from flask import Flask, render_template, request, redirect, url_for, abort
from werkzeug.utils import secure_filename
app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 1024 * 1024
app.config['UPLOAD_EXTENSIONS'] = ['.jpg', '.png', '.gif']
app.config['UPLOAD_PATH'] = 'uploads'
def validate_image(stream):
    header = stream.read(512)
    stream.seek(0) 
    format = imghdr.what(None, header)
    if not format:
        return None
    return '.' + (format if format != 'jpeg' else 'jpg')
@app.route('/')
def index():
    return render_template('index.html')
@app.route('/', methods=['POST'])
def upload_files():
    uploaded_file = request.files['file']
    filename = secure_filename(uploaded_file.filename)
    if filename != '':
        file_ext = os.path.splitext(filename)[1]
        if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                file_ext != validate_image(uploaded_file.stream):
            abort(400)
        uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))
    return redirect(url_for('index'))
```


在视图函数中唯一的变化就是加入了最后一个验证逻辑:
```python
			if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                file_ext != validate_image(uploaded_file.stream):
            abort(400)
```


这个扩展检查首先确保文件扩展名在允许的列表中，然后确保通过查看数据流检测到的文件扩展名与文件扩展名相同。

在测试这个版本的应用程序之前，创建一个名为 _uploads_ 的目录(或者你在 `UPLOAD_PATH`  配置变量中定义的路径) ，以便可以将文件保存在那里。
## 使用上传的文件
你现在知道如何处理文件上传。对于某些应用程序，这就是所需要的全部内容，因为这些文件用于某些内部进程。但是对于大量的应用程序，特别是那些具有社交功能的应用程序，比如头像，用户上传的文件必须与应用程序集成。以 `avatar` 为例，一旦用户上传了他们的 avatars 图片，任何提到用户名的地方都需要上传的图片显示在侧面。

我将文件上传分为两大类，具体取决于用户上传的文件是供公众使用还是对每个用户私有。 本文中多次讨论过的 `avatar` 图像显然属于第一类，因为这些 `avatar` 旨在与其他用户公开共享。 另一方面，对上传的图像执行编辑操作的应用程序可能在第二类中，因为你希望每个用户只能访问自己的图像。


### 公共文件上传
当图像属于公共性质时，使图像可供应用程序使用的最简单方法是将上传目录放在应用程序的静态文件夹中。例如，可以在 static 中创建 _avatars_ 子目录，然后使用用户 id 作为名称在该位置保存头像。


使用 url_for() 函数以与应用程序的常规静态文件相同的方式引用存储在静态文件夹的子目录中的这些上传文件。 我之前建议在保存上传的头像图像时使用用户 id 作为文件名。这就是图片保存的方式:
```python
uploaded_file.save(os.path.join('static/avatars', current_user.get_id()))
```


使用这个实现，给定一个用户 id，可以生成用户头像的 URL 如下:
```python
url_for('static', filename='avatars/' + str(user_id))
```


或者，可以将上传保存到静态文件夹外的目录中，然后可以添加新的路由来为其提供服务。在示例 app.py 应用程序文件中，上传的文件保存到 `UPLOAD_PATH`  配置变量中设置的位置。为了从该位置提供这些文件，我们可以实现以下路由:
```python
from flask import send_from_directory
@app.route('/uploads/<filename>')
def upload(filename):
    return send_from_directory(app.config['UPLOAD_PATH'], filename)
```


这个解决方案比在静态文件夹中存储上传的一个优点是，在返回这些文件之前，你可以实现额外的限制，要么直接在函数体内使用 Python 逻辑，要么使用 decorator。例如，如果你只希望向登录的用户提供对上传的访问，那么你可以将 Flask-Login 的 @login_required 装饰器添加到这个路由中，或者添加用于正常路由的任何其他身份验证或角色检查机制。

让我们使用这种实现思想在示例应用程序中显示上传的文件。下面是 app.py 的一个新的完整版本:
```python
import imghdr
import os
from flask import Flask, render_template, request, redirect, url_for, abort, \
    send_from_directory
from werkzeug.utils import secure_filename
app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 1024 * 1024
app.config['UPLOAD_EXTENSIONS'] = ['.jpg', '.png', '.gif']
app.config['UPLOAD_PATH'] = 'uploads'
def validate_image(stream):
    header = stream.read(512)  # 512 bytes should be enough for a header check
    stream.seek(0)  # reset stream pointer
    format = imghdr.what(None, header)
    if not format:
        return None
    return '.' + (format if format != 'jpeg' else 'jpg')
@app.route('/')
def index():
    files = os.listdir(app.config['UPLOAD_PATH'])
    return render_template('index.html', files=files)
@app.route('/', methods=['POST'])
def upload_files():
    uploaded_file = request.files['file']
    filename = secure_filename(uploaded_file.filename)
    if filename != '':
        file_ext = os.path.splitext(filename)[1]
        if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                file_ext != validate_image(uploaded_file.stream):
            abort(400)
        uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))
    return redirect(url_for('index'))
@app.route('/uploads/<filename>')
def upload(filename):
    return send_from_directory(app.config['UPLOAD_PATH'], filename)
```


除了新的 upload() 函数之外，index() 视图函数使用 os.listdir ()获取上传位置中的文件列表，并将其发送到模板以进行呈现。更新后的 _index.html _模板内容如下:
```html
<!doctype html>
<html>
  <head>
    <title>File Upload</title>
  </head>
  <body>
    <h1>File Upload</h1>
    <form method="POST" action="" enctype="multipart/form-data">
      <p><input type="file" name="file"></p>
      <p><input type="submit" value="Submit"></p>
    </form>
    <hr>
    {% for file in files %}
      <img src="{{ url_for('upload', filename=file) }}" style="width: 64px">
    {% endfor %}
  </body>
</html>
```


有了这些改变，每次你上传一张图片，页面底部就会添加一个缩略图:
![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1602229806189-449b0d88-ebdd-42f2-9494-4f4c82c92332.png)
### 私有文件上传
当用户将私有文件上传到应用程序时，需要进行额外的检查，以防止一个用户与未经授权的方共享文件。这些情况的解决方案需要上面所示的 upload() 视图函数的变体，以及额外的访问检查。

一个常见的要求是只与所有者共享上传的文件。当存在此需求时，存储上传的一种方便方法是为每个用户使用一个单独的目录。例如，可以将给定用户的上传保存到 uploads/<user_id> 目录，然后可以修改 `uploads()`  函数，使其只服务于用户自己的上传目录，这样一来，一个用户就不可能从另一个用户那里查看文件。下面你可以看到这个技术的一个可能的实现，再次假设使用了 Flask-Login:

```python
@app.route('/uploads/<filename>')
@login_required
def upload(filename):
    return send_from_directory(os.path.join(
        app.config['UPLOAD_PATH'], current_user.get_id()), filename)
```


## 显示上传进度
到目前为止，我们一直依赖 web 浏览器提供的原生文件上传小部件来启动我们的文件上传。我相信我们都同意这个小工具不是很吸引人。不仅如此，由于缺少上传进度显示，它无法用于上传大文件，因为用户在整个上传过程中不能收到任何反馈。虽然本文的范围是涵盖服务器端，但我认为，如果能够给你提供一些关于如何实现一个基于 JavaScript 的现代文件上传小部件以显示上传进度的想法将很有用。

好消息是，在服务器上不需要进行任何大的更改，无论你在浏览器中使用哪种方法来启动上传，上传机制都以相同的方式工作。 为了向你展示一个示例实现，我将用与流行的文件上传客户端 [dropzone.js](https://www.dropzonejs.com/) 兼容的index.html 替换HTML表单。


下面是一个新版本的 templates/index.html，它从 CDN 加载下拉区 CSS 和 JavaScript 文件，并根据[dropzone documentation](https://www.dropzonejs.com/#usage) 实现一个上传表单:
```html
<html>
  <head>
    <title>File Upload</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/dropzone/5.7.1/min/dropzone.min.css">
  </head>
  <body>
    <h1>File Upload</h1>
    <form action="{{ url_for('upload_files') }}" class="dropzone">
    </form>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/dropzone/5.7.1/min/dropzone.min.js"></script>
  </body>
</html>
```


在实现 dropzone 时，我发现了一件有趣的事情，那就是它要求设置 <form> 元素中的 action 属性，即使标准 forms 接受一个空的动作来指示提交到相同的 URL。

用这个新版的模板启动服务器，你会得到以下结果:
![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1602228140121-0b8d191c-52d6-46be-bde2-74ea2c80d655.png)
基本上就是这样！现在你可以拖拽文件，它们将上传到服务器，并带有一个进度条和成功或失败的最终指示。

如果文件上传失败，无论是由于文件太大或无效，dropzone 想要显示一个错误消息。因为我们的服务器正在返回413和400错误的标准 Flask 错误页面，您将在错误弹出窗口中看到一些乱七八糟的 HTML。为了纠正这个错误，我们可以更新服务器以文本形式返回错误响应。

当请求有效负载大于配置中设置的大小时，Flask 会生成文件过大条件的413错误。要覆盖默认的错误页面，我们必须使用 `app.errorhandler` 装饰器:
```python
@app.errorhandler(413)
def too_large(e):
    return "File is too large", 413
```


当任何验证检查失败时，应用程序将生成第二个错误条件。在这种情况下，错误是通过一个 `abort(400)` 调用生成的。取而代之的是，可以直接生成响应:
```python
		if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                file_ext != validate_image(uploaded_file.stream):
            return "Invalid image", 400
```


我要做的最后一个改变并不是真的需要，但是它节省了一点带宽。对于成功上传，服务器返回一个 redirect() 到主路由。这将导致上传表单再次显示，并刷新页面底部的上载缩略图列表。现在这些都不需要了，因为上传是作为后台请求通过 dropzone 完成的，所以我们可以消除重定向，并使用代码204切换到空响应。

下面是 app.py 的完整更新版本，可以与 dropzone.js 一起使用:
```python
import imghdr
import os
from flask import Flask, render_template, request, redirect, url_for, abort, \
    send_from_directory
from werkzeug.utils import secure_filename
app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 2 * 1024 * 1024
app.config['UPLOAD_EXTENSIONS'] = ['.jpg', '.png', '.gif']
app.config['UPLOAD_PATH'] = 'uploads'
def validate_image(stream):
    header = stream.read(512)
    stream.seek(0)
    format = imghdr.what(None, header)
    if not format:
        return None
    return '.' + (format if format != 'jpeg' else 'jpg')
@app.errorhandler(413)
def too_large(e):
    return "File is too large", 413
@app.route('/')
def index():
    files = os.listdir(app.config['UPLOAD_PATH'])
    return render_template('index.html', files=files)
@app.route('/', methods=['POST'])
def upload_files():
    uploaded_file = request.files['file']
    filename = secure_filename(uploaded_file.filename)
    if filename != '':
        file_ext = os.path.splitext(filename)[1]
        if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
                file_ext != validate_image(uploaded_file.stream):
            return "Invalid image", 400
        uploaded_file.save(os.path.join(app.config['UPLOAD_PATH'], filename))
    return '', 204
@app.route('/uploads/<filename>')
def upload(filename):
    return send_from_directory(app.config['UPLOAD_PATH'], filename)
```


用这个更新重新启动应用程序，现在错误将会有一个正确的消息:
![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1602228541419-b3db7180-8878-49ad-8e2b-791634cd69a8.png)
dropzone.js 库非常灵活，有许多定制选项，因此我鼓励你访问他们的文档，了解如何使其适应你的需求。你也可以寻找其他的 JavaScript 文件上传库，因为它们都遵循 HTTP 标准，这意味着你的 Flask 服务器可以很好地与它们一起工作。

翻译
[Handling File Uploads With Flask](https://blog.miguelgrinberg.com/post/handling-file-uploads-with-flask)