# 馃攲 Bolt.new + WebContainer Integration Examples

**袩芯谢薪褘械 锌褉懈屑械褉褘 锌芯写泻谢褞褔械薪懈褟 WebContainer 泻 褌胁芯械屑褍 Bolt.new**

---

## 馃摝 袙邪褉懈邪薪褌 1: 袘邪蟹芯胁芯械 锌芯写泻谢褞褔械薪懈械 WebContainer

```html
<!DOCTYPE html>
<html>
<head>
    <title>Bolt with WebContainer</title>
    <script src="https://webcontainers.io/api"></script>
</head>
<body>

<div id="editor"></div>
<div id="preview"></div>

<script>
    import { WebContainer } from '@webcontainers/api';

    let webcontainerInstance;

    async function initContainer() {
        // 袠薪懈褑懈邪谢懈蟹懈褉褍泄 WebContainer
        webcontainerInstance = await WebContainer.boot();
        console.log('WebContainer started!');
    }

    async function runCode(htmlCode, cssCode, jsCode) {
        // 小芯蟹写邪泄 褎邪泄谢褘
        await webcontainerInstance.fs.mkdir('app', { recursive: true });
        
        await webcontainerInstance.fs.writeFile(
            'app/index.html',
            htmlCode
        );

        if (cssCode) {
            await webcontainerInstance.fs.writeFile(
                'app/style.css',
                cssCode
            );
        }

        if (jsCode) {
            await webcontainerInstance.fs.writeFile(
                'app/script.js',
                jsCode
            );
        }

        // 袟邪锌褍褋褌懈 锌褉芯褋褌芯泄 HTTP 褋械褉胁械褉
        const process = await webcontainerInstance.spawn('npx', [
            'http-server',
            'app',
            '-p', '3000',
            '-o'
        ]);

        console.log('Server running on port 3000');
    }

    // 袠薪懈褑懈邪谢懈蟹邪褑懈褟 锌褉懈 蟹邪谐褉褍蟹泻械
    window.addEventListener('load', initContainer);
</script>

</body>
</html>
```

---

## 馃幆 袙邪褉懈邪薪褌 2: 袩芯谢薪邪褟 懈薪褌械谐褉邪褑懈褟 褋 Bolt.new

袛芯斜邪胁褜 褝褌芯褌 泻芯写 胁 `bolt-full-integration.html` 胁 褎褍薪泻褑懈褞 `sendPrompt()`:

```javascript
// 袛芯斜邪胁褜 胁 薪邪褔邪谢芯 褎邪泄谢邪
let webcontainerInstance = null;

// 袠薪懈褑懈邪谢懈蟹懈褉褍泄 WebContainer 锌褉懈 蟹邪谐褉褍蟹泻械
async function initWebContainer() {
    try {
        const { WebContainer } = await import('@webcontainers/api');
        webcontainerInstance = await WebContainer.boot();
        console.log('鉁� WebContainer initialized');
        updateAPIStatus(); // 袨斜薪芯胁懈 褋褌邪褌褍褋
    } catch (error) {
        console.error('鉂� WebContainer error:', error);
    }
}

// 袙褘蟹芯胁懈 锌褉懈 蟹邪谐褉褍蟹泻械 褋褌褉邪薪懈褑褘
document.addEventListener('DOMContentLoaded', async () => {
    // ... 褋褍褖械褋褌胁褍褞褖懈泄 泻芯写 ...
    await initWebContainer();
});

// 袦芯写懈褎懈褑懈褉褍泄 sendPrompt() 褎褍薪泻褑懈褞:
async function sendPrompt() {
    const input = document.getElementById('prompt-input');
    const prompt = input.value.trim();

    if (!prompt) return;
    if (!config.apiKey) {
        showMessage('鈿狅笍 Please add your API key in Settings', 'error');
        navigateTo('settings');
        return;
    }

    showMessage('馃攧 Processing your request...', 'info');
    input.value = '';

    try {
        // 袩芯谢褍褔懈 泻芯写 芯褌 AI
        const aiCode = await callAI(prompt);
        
        // 袙褘锌芯谢薪懈 胁 WebContainer
        if (webcontainerInstance) {
            await executeInWebContainer(aiCode);
            showMessage('鉁� Code executed in WebContainer!', 'success');
        } else {
            showMessage('鈿狅笍 WebContainer not available', 'error');
        }
        
        console.log('AI Response:', aiCode);
    } catch (error) {
        showMessage('鉂� Error: ' + error.message, 'error');
    }
}

// 袛芯斜邪胁褜 褎褍薪泻褑懈褞 写谢褟 胁褘锌芯谢薪械薪懈褟 泻芯写邪
async function executeInWebContainer(aiCode) {
    // 袩邪褉褋懈 泻芯写 (锌褉械写锌芯谢邪谐邪泄 褔褌芯 AI 胁械褉薪褍谢 HTML)
    const { html, css, js } = extractCode(aiCode);

    // 小芯蟹写邪泄 褋褌褉褍泻褌褍褉褍
    await webcontainerInstance.fs.mkdir('app', { recursive: true });

    // 袧邪锌懈褕懈 HTML 褎邪泄谢
    const htmlContent = html || '<h1>Hello from Bolt.new</h1>';
    await webcontainerInstance.fs.writeFile('app/index.html', htmlContent);

    // 袧邪锌懈褕懈 CSS 械褋谢懈 械褋褌褜
    if (css) {
        await webcontainerInstance.fs.writeFile('app/style.css', css);
    }

    // 袧邪锌懈褕懈 JS 械褋谢懈 械褋褌褜
    if (js) {
        await webcontainerInstance.fs.writeFile('app/script.js', js);
    }

    // 袟邪锌褍褋褌懈 褋械褉胁械褉
    await startServer();

    // 袨褌芯斜褉邪蟹懈 胁 iframe
    displayPreview();
}

// 肖褍薪泻褑懈褟 写谢褟 锌邪褉褋懈薪谐邪 泻芯写邪 芯褌 AI
function extractCode(aiResponse) {
    // 袠褖懈 HTML 斜谢芯泻懈
    const htmlMatch = aiResponse.match(/<html[\s\S]*?<\/html>/i) ||
                      aiResponse.match(/<body[\s\S]*?<\/body>/i) ||
                      aiResponse.match(/<div[\s\S]*?<\/div>/i);
    
    const cssMatch = aiResponse.match(/<style[\s\S]*?<\/style>/i);
    const jsMatch = aiResponse.match(/<script[\s\S]*?<\/script>/i);

    let html = htmlMatch ? htmlMatch[0] : '';
    let css = cssMatch ? cssMatch[0].replace(/<\/?style[^>]*>/g, '') : '';
    let js = jsMatch ? jsMatch[0].replace(/<\/?script[^>]*>/g, '') : '';

    return { html, css, js };
}

// 袟邪锌褍褋褌懈 HTTP 褋械褉胁械褉
let serverProcess = null;

async function startServer() {
    // 袨褋褌邪薪芯胁懈 褋褌邪褉褘泄 锌褉芯褑械褋褋 械褋谢懈 械褋褌褜
    if (serverProcess) {
        serverProcess.kill();
    }

    // 袟邪锌褍褋褌懈 薪芯胁褘泄
    serverProcess = await webcontainerInstance.spawn('npx', [
        'http-server',
        'app',
        '-p', '3000',
        '-c-1'
    ]);

    console.log('Server started on port 3000');
}

// 袨褌芯斜褉邪蟹懈 胁 iframe
function displayPreview() {
    let iframe = document.getElementById('preview-frame');
    
    if (!iframe) {
        iframe = document.createElement('iframe');
        iframe.id = 'preview-frame';
        iframe.style.cssText = `
            width: 100%;
            height: 500px;
            border: 1px solid #ccc;
            border-radius: 8px;
            margin-top: 20px;
        `;
        document.querySelector('.hero-section').appendChild(iframe);
    }

    // 袞写懈 锌芯泻邪 褋械褉胁械褉 蟹邪锌褍褋褌懈褌褋褟
    setTimeout(() => {
        iframe.src = 'http://localhost:3000';
    }, 1000);
}
```

---

## 馃帹 袙邪褉懈邪薪褌 3: 小 锌褉械写锌褉芯褋屑芯褌褉芯屑 褉褟写芯屑

```html
<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; height: 600px;">
    <!-- Editor -->
    <div>
        <h3>Code Editor</h3>
        <textarea id="code-input" style="width: 100%; height: 100%; font-family: monospace;"></textarea>
    </div>

    <!-- Preview -->
    <div>
        <h3>Preview</h3>
        <iframe id="preview" style="width: 100%; height: 100%; border: 1px solid #ccc;"></iframe>
    </div>
</div>

<script>
    // 袨斜薪芯胁懈 preview 锌褉懈 懈蟹屑械薪械薪懈懈 泻芯写邪
    document.getElementById('code-input').addEventListener('input', (e) => {
        const code = e.target.value;
        const iframe = document.getElementById('preview');
        
        // 袠褋锌芯谢褜蟹褍泄 Blob 写谢褟 斜械蟹芯锌邪褋薪芯谐芯 胁褘锌芯谢薪械薪懈褟
        const blob = new Blob([code], { type: 'text/html' });
        iframe.src = URL.createObjectURL(blob);
    });
</script>
```

---

## 馃攧 袙邪褉懈邪薪褌 4: 小 褋芯褏褉邪薪械薪懈械屑 褎邪泄谢芯胁

```javascript
// 肖褍薪泻褑懈褟 写谢褟 褋芯褏褉邪薪械薪懈褟 锌褉芯械泻褌邪
async function saveProject(projectName) {
    const project = {
        name: projectName,
        files: {
            'index.html': await webcontainerInstance.fs.readFile('app/index.html', 'utf-8'),
            'style.css': await webcontainerInstance.fs.readFile('app/style.css', 'utf-8').catch(() => ''),
            'script.js': await webcontainerInstance.fs.readFile('app/script.js', 'utf-8').catch(() => ''),
        },
        timestamp: new Date().toISOString()
    };

    // 小芯褏褉邪薪懈 胁 localStorage
    localStorage.setItem('project_' + projectName, JSON.stringify(project));
    console.log('Project saved:', projectName);
}

// 肖褍薪泻褑懈褟 写谢褟 蟹邪谐褉褍蟹泻懈 锌褉芯械泻褌邪
async function loadProject(projectName) {
    const project = JSON.parse(localStorage.getItem('project_' + projectName));
    
    if (!project) {
        console.error('Project not found');
        return;
    }

    // 袧邪锌懈褕懈 褎邪泄谢褘
    for (const [filename, content] of Object.entries(project.files)) {
        if (content) {
            await webcontainerInstance.fs.writeFile('app/' + filename, content);
        }
    }

    // 袩械褉械蟹邪谐褉褍蟹懈
    displayPreview();
    console.log('Project loaded:', projectName);
}

// 肖褍薪泻褑懈褟 写谢褟 褝泻褋锌芯褉褌邪 锌褉芯械泻褌邪
function exportProject(projectName) {
    const project = JSON.parse(localStorage.getItem('project_' + projectName));
    const dataStr = JSON.stringify(project, null, 2);
    
    // 小泻邪褔邪泄 泻邪泻 JSON
    const dataBlob = new Blob([dataStr], { type: 'application/json' });
    const url = URL.createObjectURL(dataBlob);
    const link = document.createElement('a');
    link.href = url;
    link.download = projectName + '.json';
    link.click();
}
```

---

## 馃搳 袙邪褉懈邪薪褌 5: 小 屑械薪械写卸屑械薪褌芯屑 褎邪泄谢芯胁

```javascript
class ProjectManager {
    constructor() {
        this.projects = new Map();
    }

    async createFile(path, content) {
        const dirs = path.split('/').slice(0, -1);
        let currentPath = '';
        
        for (const dir of dirs) {
            currentPath += dir + '/';
            await webcontainerInstance.fs.mkdir(currentPath, { recursive: true });
        }
        
        await webcontainerInstance.fs.writeFile(path, content);
    }

    async readFile(path) {
        return await webcontainerInstance.fs.readFile(path, 'utf-8');
    }

    async listFiles(path = '/') {
        const files = await webcontainerInstance.fs.readdir(path);
        return files;
    }

    async deleteFile(path) {
        await webcontainerInstance.fs.rm(path);
    }

    async renameFile(oldPath, newPath) {
        const content = await this.readFile(oldPath);
        await this.deleteFile(oldPath);
        await this.createFile(newPath, content);
    }

    async importProject(zipFile) {
        // Implement ZIP extraction
    }

    async exportProject(path) {
        // Implement ZIP creation
    }
}

const projectManager = new ProjectManager();

// 袠褋锌芯谢褜蟹芯胁邪薪懈械
await projectManager.createFile('app/index.html', '<h1>Hello</h1>');
await projectManager.createFile('app/style.css', 'h1 { color: blue; }');
```

---

## 馃殌 袙邪褉懈邪薪褌 6: 小 褍褋褌邪薪芯胁泻芯泄 蟹邪胁懈褋懈屑芯褋褌械泄

```javascript
async function installDependencies(packages) {
    showMessage('馃摝 Installing dependencies...', 'info');

    // 小薪邪褔邪谢邪 懈薪懈褑懈邪谢懈蟹懈褉褍泄 npm
    await webcontainerInstance.spawn('npm', ['init', '-y']);

    // 袟邪褌械屑 褍褋褌邪薪芯胁懈 锌邪泻械褌褘
    for (const pkg of packages) {
        console.log(`Installing ${pkg}...`);
        const process = await webcontainerInstance.spawn('npm', ['install', pkg]);
        
        process.output.pipeTo(new WritableStream({
            write(chunk) {
                console.log(chunk);
            }
        }));
    }

    showMessage('鉁� Dependencies installed!', 'success');
}

// 袠褋锌芯谢褜蟹芯胁邪薪懈械
await installDependencies(['react', 'react-dom', 'axios']);
```

---

## 馃寪 袙邪褉懈邪薪褌 7: 小 live reloading

```javascript
async function setupLiveReload() {
    // 袠褋锌芯谢褜蟹褍泄 chokidar 写谢褟 芯褌褋谢械卸懈胁邪薪懈褟 褎邪泄谢芯胁
    const watchProcess = await webcontainerInstance.spawn('npx', [
        'chokidar',
        'app/**/*',
        '-c',
        'echo "File changed"'
    ]);

    watchProcess.output.pipeTo(new WritableStream({
        write(chunk) {
            console.log('File changed, reloading...');
            // 袩械褉械蟹邪谐褉褍蟹懈 iframe
            document.getElementById('preview').src = document.getElementById('preview').src;
        }
    }));
}
```

---

## 馃敆 袙邪褉懈邪薪褌 8: 小 锌芯写泻谢褞褔械薪懈械屑 Cloudflare Workers

```javascript
async function deployToCloudflare(projectName) {
    // 袠薪懈褑懈邪谢懈蟹懈褉褍泄 Wrangler
    await webcontainerInstance.spawn('npm', ['init', '-y']);
    await webcontainerInstance.spawn('npm', ['install', '-D', 'wrangler']);

    // 小芯蟹写邪泄 wrangler.toml
    const wranglerConfig = `
name = "${projectName}"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[env.production]
routes = [
  { pattern = "yoursite.com/*", zone_name = "yoursite.com" }
]
`;

    await webcontainerInstance.fs.writeFile('wrangler.toml', wranglerConfig);

    // 袛械锌谢芯泄
    const deployProcess = await webcontainerInstance.spawn('npx', [
        'wrangler',
        'deploy'
    ]);

    console.log('Deploying to Cloudflare...');
}
```

---

## 馃摑 袩芯谢薪褘泄 锌褉懈屑械褉 胁 芯写薪芯屑 褎邪泄谢械

```javascript
class BoltWebContainerIntegration {
    constructor() {
        this.container = null;
        this.serverProcess = null;
        this.projects = new Map();
    }

    // 袠薪懈褑懈邪谢懈蟹邪褑懈褟
    async init() {
        const { WebContainer } = await import('@webcontainers/api');
        this.container = await WebContainer.boot();
        console.log('鉁� WebContainer ready');
    }

    // 袟邪锌褍褋泻 泻芯写邪
    async runCode(html, css = '', js = '') {
        await this.container.fs.mkdir('app', { recursive: true });
        await this.container.fs.writeFile('app/index.html', this.buildHTML(html, css, js));

        if (css) {
            await this.container.fs.writeFile('app/style.css', css);
        }

        if (js) {
            await this.container.fs.writeFile('app/script.js', js);
        }

        await this.startServer();
    }

    buildHTML(html, css, js) {
        return `
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bolt Preview</title>
    <style>${css}</style>
</head>
<body>
    ${html}
    <script>${js}</script>
</body>
</html>
        `;
    }

    async startServer() {
        if (this.serverProcess) this.serverProcess.kill();
        this.serverProcess = await this.container.spawn('npx', [
            'http-server', 'app', '-p', '3000', '-c-1'
        ]);
    }

    async saveProject(name, html, css, js) {
        this.projects.set(name, { html, css, js, timestamp: Date.now() });
        localStorage.setItem('bolt_projects', JSON.stringify(Array.from(this.projects.entries())));
    }

    loadProject(name) {
        return this.projects.get(name);
    }

    listProjects() {
        return Array.from(this.projects.keys());
    }
}

// 袠褋锌芯谢褜蟹芯胁邪薪懈械
const bolt = new BoltWebContainerIntegration();
await bolt.init();
await bolt.runCode('<h1>Hello World</h1>', 'h1 { color: blue; }', '');
```

---

## 馃悰 袪械褕械薪懈械 褔邪褋褌褘褏 锌褉芯斜谢械屑

### CORS 芯褕懈斜泻懈
```javascript
// 袠褋锌芯谢褜蟹褍泄 CORS proxy 写谢褟 API 蟹邪锌褉芯褋芯胁
const corsProxy = 'https://cors-anywhere.herokuapp.com/';
```

### WebContainer 薪械 蟹邪谐褉褍卸邪械褌褋褟
```javascript
// 袩褉芯胁械褉褜 懈薪褌械褉薪械褌 懈 斜褉邪褍蟹械褉 锌芯写写械褉卸懈胁邪械褌 SharedArrayBuffer
if (!window.SharedArrayBuffer) {
    console.error('SharedArrayBuffer not supported');
}
```

### 袩芯褉褌 3000 蟹邪薪褟褌
```javascript
// 袠褋锌芯谢褜蟹褍泄 写褉褍谐芯泄 锌芯褉褌
await spawn('http-server', ['app', '-p', '3001']);
```

---

## 馃摎 袛芯泻褍屑械薪褌邪褑懈褟

- **WebContainer API:** https://webcontainers.io/api
- **NPM in WebContainer:** https://webcontainers.io/guides/nodejs
- **肖邪泄谢芯胁邪褟 褋懈褋褌械屑邪:** https://webcontainers.io/guides/filesystem

---

袙褘斜械褉懈 薪褍卸薪褘泄 胁邪褉懈邪薪褌 懈 写芯斜邪胁褜 胁 褋胁芯泄 Bolt.new! 馃殌
