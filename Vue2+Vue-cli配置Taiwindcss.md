### Vue2+Vue-cli配置Taiwindcss

1. 安装Tailwind相关依赖

   `npm install tailwindcss@npm:@tailwindcss/postcss7-compat @tailwindcss/postcss7-compat postcss@^7 autoprefixer@^9`

2. 生成Tailwindcss默认配置文件tailwind.config.js和postcss.config.js

   `npx tailwindcss init`

3. 创建tailwindcss样式文件并在main.js引入

   ```css
   @import "tailwindcss/base";
   @import "tailwindcss/components";
   @import "tailwindcss/utilities";
   ```

   ```javascript
   import './assets/style/tailwindcss.css'
   ```

