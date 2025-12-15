## 项目概述

### 项目背景

随着高校教学活动与校园生活的持续进行，教材、辅导资料与课外读物在学期末与专业调整时常被闲置。传统的线下交换信息零散、效率低且缺乏信任保障；而通用二手平台缺乏面向校园的实名认证与配送闭环，无法很好满足校园场景的特殊需求。

为缓解教材浪费、降低学术资源获取成本并构建校园内部循环生态，本项目拟设计并实现一套**面向高校的二手书交易平台——“书屋（Campus Book Trade）”**，为师生提供便捷、安全、高效的书籍流通渠道。

### 项目名称

**项目名称：书屋（Campus Book Trade）**

### 项目定位

“书屋”定位为校园内**C2C 为主、辅以 B2C 的混合型电子商务平台**，以学生二手书交易为核心，集成学生实名认证、校园社交、校内配送与智能推荐等功能，目标成为校园内可信赖的书籍流通与知识共享平台。

### **运行步骤**

安装依赖包：

```bash
pip install -r requirements.txt
```

配置数据库：

创建 MySQL 数据库 python_mask
在 mask/settings.py 中配置数据库连接信息
恢复数据库结构：
```bash
python manage.py migrate
```

创建管理员账户（可选）：

```bash
python manage.py createsuperuser
```

运行项目：

```bash
python manage.py runserver
```

访问应用：

前台页面：http://127.0.0.1:8000/
管理后台：http://127.0.0.1:8000/admin/

### **项目结构说明**
```
mask/                 # Django项目配置文件夹
├── settings.py       # 项目配置文件
├── urls.py           # 主URL配置
└── wsgi.py           # WSGI部署配置

app/                  # 主应用文件夹
├── models.py         # 数据模型定义
├── views.py          # 视图处理逻辑
├── forms.py          # 表单处理
├── urls.py           # 应用URL配置
├── admin.py          # 后台管理配置
└── migrations/       # 数据库迁移文件

templates/            # 模板文件夹
├── app/              # 应用模板
└── base/             # 基础模板

static/               # 静态文件夹
├── css/              # 样式文件
├── img/              # 图片资源
└── js/               # JavaScript文件
```
### **主要功能模块**

1. 首页 (Index)
 - 展示所有已审核通过的二手书供需信息
 - 支持按照供应 / 需求类型筛选信息
 - 支持关键词搜索（匹配书籍标题）
 - 分页显示信息列表（每页 15 条）
 - 显示信息基本信息（标题、发布时间、类型标签等）
2. 信息发布 (Commit)
 - 仅登录用户可访问，提供表单填写供需信息
 - 需填写字段：标题（必填）、类型（供应 / 需求，必填）、有效期（1 天 / 3 天 / 7 天，必填）、所在地（必填）、联系人（必填）、手机号（必填）、微信（选填）
 - 集成防滥用机制：通过django-ratelimit限制单 IP 每小时最多提交 100 次
 - 发布后需后台审核通过才会展示在首页
3. 详情页 (Detail)
 - 显示单条供需信息的完整内容（标题、类型、发布时间、有效期、所在地、联系人、电话、微信等）
 - 提供 “返回列表” 快捷操作
 - 记录信息浏览量（PV）
4. 后台管理
 - 通过 Django Admin 管理所有供需信息
 - 支持审核操作（修改状态为 “已审核”/“待审核”）
 - 支持编辑、删除信息
 - 查看信息发布时间、浏览量等数据




### **数据模型**

主要数据模型为 Product，包含以下字段：

title: 标题
type: 类型（供应/需求）
contact: 联系人
phone: 手机号
weixin: 微信号
location: 所在地
expire: 有效期（天）
status: 状态（是否审核通过）
timestamp: 发布时间

### **自定义模板标签**

项目包含自定义模板标签 app_tag：

- time_since: 显示相对发布时间
- check_expire: 检查信息是否过期

### **部署注意事项**

- 修改 mask/settings.py 中的 SECRET_KEY
- 设置 DEBUG = False 用于生产环境
- 配置合适的数据库连接
- 配置静态文件收集和部署

### 启动项目 

```
django-admin startproject mask
```

### 创建应用

```
python3 manage.py startapp app
```

### model设计

主要是对需求表Product进行设计，在此项目中，我们需要标题、联系人、电话等字段。可参考models.py文件。

设计字段如下：

```python
class Product(models.Model):
    list_display = ("title", "type", "location")
    title = models.CharField(max_length=100,blank=True, null=True)
    type = models.IntegerField(default=0)
    pv = models.IntegerField(default=0)
    contact = models.CharField(max_length=10,blank=True, null=True)
    location = models.CharField(max_length=20,blank=True, null=True)
    phone = models.CharField(max_length=13, blank=True, null=True)
    weixin = models.CharField(max_length=50, blank=True, null=True)
    status = models.BooleanField(default=False)
    timestamp = models.DateTimeField(auto_now_add=True, null=True)
    expire = models.IntegerField(default=1)
```


### 业务编写
本项目一共分为4个主要页面：首页（列表页）、详情页、发布页、登录/注册页。
Model设计
主要是对需求表Product进行设计，在此项目中，我们需要标题、联系人、电话等字段。可参考models.py文件。
设计字段如下：

```python 
class Product(models.Model):
    list_display = ("title", "type", "location")
    title = models.CharField(max_length=100,blank=True, null=True)
    type = models.IntegerField(default=0)  # 0=供应, 1=需求
    pv = models.IntegerField(default=0)
    contact = models.CharField(max_length=10,blank=True, null=True)
    location = models.CharField(max_length=20,blank=True, null=True)
    phone = models.CharField(max_length=13, blank=True, null=True)
    weixin = models.CharField(max_length=50, blank=True, null=True)
    status = models.BooleanField(default=False)  # False=待审核, True=已审核
    timestamp = models.DateTimeField(auto_now_add=True, null=True)
    expire = models.IntegerField(default=1)  # 有效期(天)

    def __str__(self):
        return self.title

    class Meta:
        db_table = "mask_product"

```

#### 首页
首先是首页，它的模版位于```templates/app/index.html```，主要是用来展示商品列表，并支持分类筛选和搜索。所有的接口都位于```app/urls.py```里面，如下：

```python 
app_name = 'app'
urlpatterns = [
    path('index', views.IndexView.as_view(), name='index'),
    path('detail/<int:pk>', views.DetailView.as_view(), name='detail'),
    path('commit', views.CommitView.as_view(), name='commit'),
    path('login/', views.CustomLoginView.as_view(), name='login'),
    path('logout/', views.CustomLogoutView.as_view(), name='logout'),
    path('register/', views.RegisterView.as_view(), name='register')
]
```

设置首页的路由为```IndexView```，开始编写```IndexView```的代码：

```python
class IndexView(generic.ListView):
    """商品列表页：支持分类筛选、搜索和分页"""
    model = Product
    template_name = 'app/index.html'
    context_object_name = 'product_list'
    paginate_by = 15
    c = None  # 分类参数（0:供应，1:需求）
    q = None  # 搜索关键词

    def get_context_data(self, object_list=None, **kwargs):
        """补充上下文：分页数据、分类参数、搜索关键词"""
        context = super().get_context_data(object_list=object_list, **kwargs)
        paginator = context.get('paginator')
        page = context.get('page_obj')
        context['page_list'] = get_page_list(paginator, page)  # 分页控件
        context['c'] = self.c  # 传递分类参数到模板
        context['q'] = self.q  # 传递搜索关键词到模板（用于回显）
        return context

    def get_queryset(self):
        """获取商品列表：支持分类筛选和搜索"""
        # 1. 获取分类参数（c）和搜索关键词（q）
        self.c = self.request.GET.get("c", None)
        self.q = self.request.GET.get("q", "").strip()  # 去除首尾空格

        # 基础查询：只显示已审核商品（status=1），按发布时间倒序
        queryset = Product.objects.filter(status=1).order_by('-timestamp')

        # 2. 应用分类筛选（如果有c参数）
        if self.c:
            queryset = queryset.filter(type=self.c)

        # 3. 应用搜索筛选（如果有关键词q）：模糊匹配标题
        if self.q:
            queryset = queryset.filter(title__icontains=self.q)

        return queryset
```
功能说明：
- 使用generic.ListView通用视图类，简化开发
- 支持双筛选：分类筛选(?c=0或?c=1)和关键词搜索(?q=关键词)
- 只显示已审核(status=1)的商品，按发布时间倒序排列
- 分页显示，每页15条数据
- 通过自定义辅助函数get_page_list实现分页控件

#### 详情页
详情页从urls.py中看到，是由```DetailView```来实现的：
```python
class DetailView(generic.DetailView):
    """商品详情页：展示单个商品的详细信息"""
    model = Product
    template_name = 'app/detail.html'
    context_object_name = 'product'  # 模板中使用的变量名

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        return context
```
功能说明：
- 继承自DetailView通用模板类来显示详情
- 使用<int:pk>捕获URL中的主键ID，自动查找对应商品
- 在模板中可以通过{{ product.title }}等方式访问商品属性
- 如需实现浏览量统计，可在get_object方法中添加pv字段自增逻辑
#### 发布页
发布页是由```CommitView```来实现的：
```python
@method_decorator(login_required, name='dispatch')
class CommitView(generic.CreateView):
    """商品发布页：仅登录用户可访问，含频率限制"""
    model = Product
    form_class = CommitForm
    template_name = 'app/commit.html'

    @ratelimit(key='ip', rate='2/m')  # 限制每IP每分钟最多2次提交
    def post(self, request, *args, **kwargs):
        was_limited = getattr(request, 'limited', False)
        if was_limited:
            messages.warning(request, "操作太频繁了，请1分钟后再试")
            return render(request, self.template_name, {'form': CommitForm()})
        return super().post(request, *args, **kwargs)

    def get_success_url(self):
        messages.success(self.request, "发布成功！您的商品已提交审核")
        return reverse('app:commit')  # 返回发布页，可继续发布
```
功能说明：
- 继承自CreateView，因为是创建操作
- 使用@method_decorator(login_required, name='dispatch')装饰器，限制只有登录用户才能访问
- 在post方法中，通过ratelimit装饰器限制提交次数，防止恶意提交（原代码为100/h，现改为2/m更合理）
- 发布成功后，商品status默认为False（待审核），需管理员审核后才会显示在首页
- 使用Django的messages框架显示操作反馈
- 
#### 用户认证

1. 登录功能
```python
class CustomLoginView(LoginView):
    """自定义登录页：指定模板，已登录用户自动跳转"""
    template_name = 'app/login.html'
    redirect_authenticated_user = True  # 已登录用户访问登录页时自动跳转

    def get_success_url(self):
        return reverse('home')  # 登录成功返回首页
```
功能说明：
- 继承Django内置的LoginView，自定义登录模板
- redirect_authenticated_user=True防止已登录用户重复登录
- 登录成功后重定向到首页

2. 注册功能
```python
class RegisterView(generic.FormView):
    """用户注册页：处理注册逻辑"""
    template_name = 'app/register.html'
    form_class = RegisterForm
    success_url = '/app/login/'  # 注册成功跳转登录页

    def form_valid(self, form):
        form.save()  # 保存新用户
        messages.success(self.request, "注册成功，请使用新账号登录")
        return super().form_valid(form)
```

功能说明：
- 使用FormView处理注册表单
- 注册成功后自动跳转到登录页
- 使用RegisterForm表单类处理用户创建逻辑

#### 辅助功能
分页辅助函数（```helpers.py```）
```python
def get_page_list(paginator, page):
    """
    生成分页控件页码列表
    :param paginator: 分页器对象
    :param page: 当前页对象
    :return: 显示的页码列表
    """
    page_list = []
    # 当前页前后各显示3页
    start = max(1, page.number - 3)
    end = min(paginator.num_pages, page.number + 3)
    
    for i in range(start, end + 1):
        page_list.append(i)
    
    # 添加首尾页和省略号
    if start > 1:
        page_list = [1, '...'] + page_list
    if end < paginator.num_pages:
        page_list = page_list + ['...', paginator.num_pages]
    
    return page_list
```
功能说明：
- 生成分页控件所需的页码列表
- 当前页前后各显示3页
- 添加首页、末页和省略号，提升用户体验

### 运行项目

```bash
python3 manage.py runserver
```

