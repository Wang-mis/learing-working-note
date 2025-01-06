# 打包前端项目
vite.config.js配置
``` js
import { fileURLToPath, URL } from 'node:url'  
  
import { defineConfig } from 'vite'  
import vue from '@vitejs/plugin-vue'  
  
export default defineConfig({  
  plugins: [  
    vue()  
  ],  resolve: {  
    alias: {  
      '@': fileURLToPath(new URL('./src', import.meta.url))  
    }  },  //本地运行配置，以及反向代理配置  
  server: {  
    // host: "localhost",  
    host: '0.0.0.0',  
    https: false,//是否启用 http 2    cors: true,//为开发服务器配置 CORS , 默认启用并允许任何源  
    // open: true,//服务启动时自动在浏览器中打开应用  
    open: false,  
    port: '5173',  
    strictPort: false, //设为true时端口被占用则直接退出，不会尝试下一个可用端口  
    force: true,//是否强制依赖预构建  
    hmr: false,//禁用或配置 HMR 连接  
    // 传递给 chockidar 的文件系统监视器选项  
    watch: {  
      ignored: ['!**/node_modules/your-package-name/**']  
    },    // 反向代理配置  
    proxy: {  
      '/api': {  
        target: 'http://0.0.0.0:14451',  
        changeOrigin: true,  
        rewrite: (path) => path.replace(/^\/api/, '')  
      },      '/biaseer': {  
        target: 'http://0.0.0.0:14452',  
        changeOrigin: true,  
        rewrite: (path) => path.replace(/^\/biaseer/, '')  
      },      '/newsum': {  
        target: 'http://0.0.0.0:14453',  
        changeOrigin: true,  
        rewrite: (path) => path.replace(/^\/newsum/, '')  
      }    }  },  //打包配置  
  build: {  
    //浏览器兼容性  "esnext"|"modules"    target: 'modules',  
    //指定输出路径  
    outDir: 'dist',  
    //生成静态资源的存放路径  
    assetsDir: 'assets',  
    //小于此阈值的导入或引用资源将内联为 base64 编码，以避免额外的 http 请求。设置为 0 可以完全禁用此项  
    assetsInlineLimit: 4096,  
    //启用/禁用 CSS 代码拆分  
    cssCodeSplit: true,  
    //构建后是否生成 source map 文件  
    sourcemap: false,  
    //自定义底层的 Rollup 打包配置  
    rollupOptions: {}  
  }})
```

``` bash
# 构建
npm run build
# 如果没有/var/www/html/<project_name>/，创建
mkdir /var/www/html/NewsInsightX/
# 删除/var/www/html/<project_name>/内的所有文件
rm -f /var/www/html/NewsInsightX/*
# 将构建后的文件移动到/var/www/html/<project_name>/中
cp -r dist/* /var/www/html/NewsInsightX/
```
# 配置Apache
``` c
/etc/httpd/conf/httpd.conf

# 在文件最后添加：
<VirtualHost *:80>
	# 域名
    ServerName newsinsight.cn

	# 打包文件位置 /var/www/html/<project_name>/
    DocumentRoot /var/www/html/NewsInsightX

    # 配置反向代理
    ProxyPass /api http://localhost:14451
    ProxyPassReverse /api http://localhost:14451

    ProxyPass /biaseer http://localhost:14452
    ProxyPassReverse /biaseer http://localhost:14452

    ProxyPass /newsum http://localhost:14453
    ProxyPassReverse /newsum http://localhost:14453

    # 允许跨域请求
    Header always set Access-Control-Allow-Origin "*"

    <Directory /var/www/html/NewsInsightX>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
单页面应用的url由前端负责解析，与Apache无关，因此需要重写url请求，将所有请求重定向到`index.html`。
在`/var/www/html/<project_name>/`内创建`.htaccess`文件。
``` c
RewriteEngine On
RewriteBase /

# 重写请求
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.html [L]
```
启动apache服务。
``` bash
systemctl start httpd 
```
