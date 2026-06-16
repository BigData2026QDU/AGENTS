# FRONTEND.md

> 前端开发规范 - 交互式 Web 应用

## 0. 关键约束（CRITICAL）

### 0.1 技术栈白名单（强制）

**禁止引入白名单之外的任何技术栈或第三方库。**

#### 允许的技术栈

| 技术 | 用途 | 说明 |
|------|------|------|
| HTML | 页面结构 | 标准 HTML5 |
| CSS | 样式设计 | 原生 CSS3，支持变量和动画 |
| JavaScript | 交互逻辑 | 原生 ES6+ |
| ECharts | 数据可视化 | 图表库 |
| Axios | HTTP 请求 | 网络请求库 |
| GSAP | 动画引擎 | 高性能动画库 |

#### 禁止使用

- ❌ 前端框架（React、Vue、Angular、Svelte 等）
- ❌ UI 组件库（Bootstrap、Element UI、Ant Design 等）
- ❌ CSS 预处理器（Sass、Less、Stylus 等）
- ❌ 构建工具（Webpack、Vite、Parcel 等）
- ❌ 其他任何未在白名单中的库

---

### 0.2 统一网络模块（强制）

**所有网络请求必须通过统一的网络模块进行，禁止直接调用 Axios。**

#### 网络模块职责

```
src/modules/network.js
    ├─ 统一请求封装
    ├─ 错误处理
    ├─ 请求拦截
    ├─ 响应拦截
    └─ 超时控制
```

#### 标准实现

```javascript
// src/modules/network.js
const NetworkModule = {
    baseURL: '', // 配置基础 URL
    timeout: 10000, // 默认超时 10 秒
    
    /**
     * 通用请求方法
     * @param {string} method - 请求方法 (GET/POST/PUT/DELETE)
     * @param {string} url - 请求路径
     * @param {Object} data - 请求数据
     * @param {Object} config - 额外配置
     * @returns {Promise}
     */
    async request(method, url, data = null, config = {}) {
        try {
            const response = await axios({
                method,
                url: this.baseURL + url,
                [method === 'GET' ? 'params' : 'data']: data,
                timeout: this.timeout,
                ...config
            });
            return { success: true, data: response.data };
        } catch (error) {
            console.error(`[Network Error] ${method} ${url}:`, error);
            return { 
                success: false, 
                error: error.message,
                code: error.response?.status 
            };
        }
    },
    
    get(url, params, config) {
        return this.request('GET', url, params, config);
    },
    
    post(url, data, config) {
        return this.request('POST', url, data, config);
    },
    
    put(url, data, config) {
        return this.request('PUT', url, data, config);
    },
    
    delete(url, params, config) {
        return this.request('DELETE', url, params, config);
    }
};

// ✅ 正确使用
const result = await NetworkModule.get('/api/data', { id: 1 });
if (result.success) {
    console.log(result.data);
}

// ❌ 禁止直接使用 axios
const response = await axios.get('/api/data'); // 不允许！
```

---

### 0.3 日间/夜间主题模块（强制）

**所有主题切换功能必须通过统一的主题模块开发。**

#### 主题模块职责

```
src/modules/theme.js
    ├─ 主题切换逻辑
    ├─ 主题状态存储
    ├─ CSS 变量管理
    └─ 主题持久化
```

#### 标准实现

```javascript
// src/modules/theme.js
const ThemeModule = {
    currentTheme: 'light', // 'light' | 'dark'
    
    /**
     * 初始化主题
     */
    init() {
        // 从本地存储读取主题设置
        const saved = localStorage.getItem('theme');
        if (saved) {
            this.setTheme(saved);
        } else {
            // 检测系统主题偏好
            const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
            this.setTheme(prefersDark ? 'dark' : 'light');
        }
    },
    
    /**
     * 设置主题
     * @param {string} theme - 'light' | 'dark'
     */
    setTheme(theme) {
        this.currentTheme = theme;
        document.documentElement.setAttribute('data-theme', theme);
        localStorage.setItem('theme', theme);
        this.applyThemeVariables(theme);
    },
    
    /**
     * 切换主题
     */
    toggle() {
        const newTheme = this.currentTheme === 'light' ? 'dark' : 'light';
        this.setTheme(newTheme);
    },
    
    /**
     * 应用主题 CSS 变量
     */
    applyThemeVariables(theme) {
        const root = document.documentElement;
        if (theme === 'dark') {
            root.style.setProperty('--bg-color', '#1a1a1a');
            root.style.setProperty('--text-color', '#ffffff');
            root.style.setProperty('--border-color', '#333333');
        } else {
            root.style.setProperty('--bg-color', '#ffffff');
            root.style.setProperty('--text-color', '#333333');
            root.style.setProperty('--border-color', '#e0e0e0');
        }
    }
};

// 初始化
document.addEventListener('DOMContentLoaded', () => {
    ThemeModule.init();
});
```

#### CSS 配合

```css
/* styles/theme.css */
:root {
    /* 日间主题（默认） */
    --bg-color: #ffffff;
    --text-color: #333333;
    --border-color: #e0e0e0;
    --primary-color: #1890ff;
}

[data-theme="dark"] {
    /* 夜间主题 */
    --bg-color: #1a1a1a;
    --text-color: #ffffff;
    --border-color: #333333;
    --primary-color: #40a9ff;
}

body {
    background-color: var(--bg-color);
    color: var(--text-color);
    transition: background-color 0.3s, color 0.3s;
}
```

---

### 0.4 Submodule 测试规范（强制）

**所有前端项目必须作为 Git Submodule 加入到前端测试仓库进行测试。**

#### 测试仓库结构

```
FrontendTestProject/
├── AGENTS/                    # 规范文档
├── tests/                     # 测试脚本
│   ├── unit/                 # 单元测试
│   ├── integration/          # 集成测试
│   └── e2e/                  # 端到端测试
├── projects/                  # 前端项目（全部为 submodule）
│   ├── ProjectA/
│   └── ProjectB/
└── README.md
```

#### 规则

1. 前端项目放在 `projects/` 目录下，每个项目独立一个 submodule
2. 禁止直接复制或嵌套其他项目代码，必须使用 submodule
3. 测试脚本引用 `projects/` 下的项目进行测试

**测试仓库地址：** (待补充)

---

## 1. 项目结构规范

### 1.1 标准目录结构

```
project-name/
├── index.html                 # 主入口页面
├── src/
│   ├── modules/              # 核心模块（必需）
│   │   ├── network.js       # 网络模块（必需）
│   │   ├── theme.js         # 主题模块（必需）
│   │   └── utils.js         # 工具模块
│   ├── components/           # 可复用组件
│   │   ├── header.js
│   │   ├── footer.js
│   │   └── ...
│   ├── pages/                # 页面逻辑
│   │   ├── home.js
│   │   └── ...
│   └── main.js               # 应用入口
├── styles/
│   ├── theme.css            # 主题样式（必需）
│   ├── base.css             # 基础样式
│   └── components/          # 组件样式
├── assets/
│   ├── images/
│   └── fonts/
├── Architecture.md           # 架构文档
├── README.md                 # 项目文档
└── File_Index.md            # 文件索引
```

---

## 2. 设计原则

### 2.1 交互式设计原则（强制）

**所有前端项目必须遵循交互式设计原则。**

#### 核心原则

1. **即时反馈：** 用户操作必须有即时的视觉反馈
2. **状态可见：** 系统状态必须清晰可见（加载中、成功、失败）
3. **操作可逆：** 关键操作支持撤销或二次确认
4. **一致性：** 相同操作在不同场景下行为一致
5. **容错性：** 提供明确的错误提示和恢复方案
6. **可访问性：** 支持键盘导航，提供语义化 HTML

#### 交互反馈示例

```javascript
// ✅ 好的实践：操作有明确反馈
async function submitForm() {
    // 1. 禁用按钮，显示加载状态
    const btn = document.getElementById('submit-btn');
    btn.disabled = true;
    btn.textContent = '提交中...';
    
    // 2. 执行请求
    const result = await NetworkModule.post('/api/submit', formData);
    
    // 3. 根据结果给反馈
    if (result.success) {
        showToast('提交成功！', 'success');
        // 成功后跳转或更新 UI
    } else {
        showToast('提交失败：' + result.error, 'error');
        btn.disabled = false;
        btn.textContent = '重新提交';
    }
}

// ❌ 不好的实践：没有反馈
async function submitForm() {
    await NetworkModule.post('/api/submit', formData);
    // 用户不知道是否成功，也不知道是否还在处理中
}
```

---

### 2.2 前端风格确定流程（强制）

**在任何前端开发开始前，必须明确项目的前端风格。**

#### 必须明确的内容

1. **色彩方案：**
   - 主色调（Primary Color）
   - 辅助色（Secondary Color）
   - 成功/警告/错误/信息色
   - 日间/夜间主题配色

2. **布局风格：**
   - 页面布局（居中/全宽/侧边栏）
   - 响应式断点
   - 栅格系统（如有）

3. **组件风格：**
   - 按钮样式（圆角/直角/扁平/立体）
   - 表单样式
   - 卡片样式
   - 导航样式

4. **动画风格：**
   - 过渡时长（快/中/慢）
   - 缓动函数（ease/linear/cubic-bezier）
   - 页面切换动画

5. **字体排版：**
   - 标题字体和大小
   - 正文字体和大小
   - 行高和字间距

#### 风格文档模板

创建 `STYLE_GUIDE.md`：

```markdown
# 项目前端风格指南

## 色彩方案
- 主色调：#1890ff
- 辅助色：#52c41a
- 成功：#52c41a
- 警告：#faad14
- 错误：#f5222d

## 布局
- 最大宽度：1200px
- 响应式断点：768px / 1024px

## 组件
- 按钮圆角：4px
- 卡片阴影：0 2px 8px rgba(0,0,0,0.1)

## 动画
- 过渡时长：0.3s
- 缓动函数：ease-in-out
```

---

## 3. 代码规范

### 3.1 JavaScript 规范

**命名规范：**
```javascript
// 变量/函数：camelCase
const userName = 'John';
function getUserInfo() { }

// 常量：UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';

// 类/构造函数：PascalCase
class DataProcessor { }

// 私有方法：下划线前缀
function _privateHelper() { }
```

**模块化规范：**
```javascript
// ✅ 使用 IIFE 避免全局污染
const MyModule = (function() {
    // 私有变量
    let privateVar = 0;
    
    // 私有方法
    function privateMethod() { }
    
    // 公共 API
    return {
        publicMethod() { },
        getData() { }
    };
})();

// ✅ 使用 ES6 模块（如果支持）
export const MyModule = {
    method1() { },
    method2() { }
};
```

**注释规范：**
```javascript
/**
 * 获取用户信息
 * 
 * @param {number} userId - 用户 ID
 * @returns {Promise<Object>} 用户信息对象
 * @throws {Error} 当用户不存在时
 */
async function getUserInfo(userId) {
    // 实现逻辑
}
```

---

### 3.2 HTML 规范

**语义化 HTML：**
```html
<!-- ✅ 好的实践：使用语义化标签 -->
<header>
    <nav>
        <ul>
            <li><a href="#home">首页</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <h1>标题</h1>
        <p>内容</p>
    </article>
</main>

<footer>
    <p>&copy; 2026</p>
</footer>

<!-- ❌ 不好的实践：全用 div -->
<div class="header">
    <div class="nav">...</div>
</div>
```

**可访问性：**
```html
<!-- ARIA 标签 -->
<button aria-label="关闭对话框" onclick="closeDialog()">×</button>

<!-- 表单标签关联 -->
<label for="username">用户名：</label>
<input id="username" type="text" name="username">

<!-- 图片替代文本 -->
<img src="logo.png" alt="公司 Logo">
```

---

### 3.3 CSS 规范

**命名规范（BEM）：**
```css
/* Block__Element--Modifier */
.card { }
.card__header { }
.card__body { }
.card--highlighted { }

/* ✅ 好的实践 */
.user-profile { }
.user-profile__avatar { }
.user-profile__name { }
.user-profile--compact { }

/* ❌ 避免：过度嵌套 */
.page .container .content .item .title { }  /* 太深 */
```

**使用 CSS 变量：**
```css
:root {
    --primary-color: #1890ff;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --border-radius: 4px;
}

.button {
    background-color: var(--primary-color);
    padding: var(--spacing-sm) var(--spacing-md);
    border-radius: var(--border-radius);
}
```

---

## 4. 性能优化

### 4.1 加载优化

```html
<!-- 1. 延迟加载非关键资源 -->
<script src="analytics.js" defer></script>

<!-- 2. 预加载关键资源 -->
<link rel="preload" href="main.css" as="style">

<!-- 3. 懒加载图片 -->
<img src="placeholder.jpg" data-src="real-image.jpg" loading="lazy">
```

### 4.2 渲染优化

```javascript
// ✅ 批量 DOM 操作
const fragment = document.createDocumentFragment();
for (let item of items) {
    const li = document.createElement('li');
    li.textContent = item;
    fragment.appendChild(li);
}
list.appendChild(fragment);

// ❌ 避免：频繁 DOM 操作
for (let item of items) {
    list.innerHTML += `<li>${item}</li>`;  // 每次都重绘
}
```

---

## 5. 测试规范

### 5.1 手动测试清单

**提交前检查：**
- [ ] 所有页面在日间/夜间主题下显示正常
- [ ] 所有交互操作有明确反馈
- [ ] 网络请求失败时有错误提示
- [ ] 响应式布局在不同屏幕尺寸下正常
- [ ] 支持键盘导航（Tab 键）
- [ ] 所有图片有 alt 属性
- [ ] 控制台无错误和警告

### 5.2 浏览器兼容性

**最低支持：**
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

---

## 6. 提交检查清单

**代码质量：**
- [ ] 只使用白名单中的技术栈
- [ ] 所有网络请求通过 NetworkModule
- [ ] 主题切换通过 ThemeModule
- [ ] 遵循交互式设计原则
- [ ] 已明确项目前端风格
- [ ] 代码格式化，无语法错误
- [ ] 关键功能有注释

**文档：**
- [ ] Architecture.md 已更新
- [ ] README.md 包含风格说明
- [ ] File_Index.md 已更新
- [ ] STYLE_GUIDE.md 已创建

**测试：**
- [ ] 已作为 submodule 加入测试仓库
- [ ] 通过手动测试清单

---

**文档版本：** 1.0  
**更新日期：** 2026-06-16  
**维护者：** xty
