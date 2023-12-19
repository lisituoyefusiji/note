# Husky+ESlint+Prettier配置代码审查

> Tminihu
>
>  2023-03-15 21:55
>
> https://zhuanlan.zhihu.com/p/614317463

#### 1. 安装依赖

```text
npm install --save-dev prettier husky lint-staged eslint
```

#### 2. 配置prettier的规则

#### 3. 配置eslint规则

#### 4. 在package.json中增加husky和lint-staged的配置

```text
"husky":{
    "hooks": {
        "pre-commit": "lint-staged"
    }
},
"lint-staged": {
    "src/**": [
        "prettier --write",
        "eslint --fix",
        "git add"
    ]
}
```

#### 5. 启用husky， 启用后，项目根目录会出现一个`.husky`文件夹

```text
npx husky install
```

#### 6. 编辑`package.json`文件，在`scripts`中添加husky命令

```text
"scripts": {
  "serve": "vue-cli-service serve",
  "build": "vue-cli-service build",
  "lint-fix": "vue-cli-service lint --fix ./src --ext .vue,.js,.ts",
  "prepare": "husky install"
},
```

#### 7. 新增 pre-commit hook

```text
npx husky add .husky/pre-commit

#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged --allow-empty
```

#### 8. 在项目根目录创建`.lintstagedrc.json`文件，在该文件中添加下面的代码：

```text
{
  "*.{ts,js,vue,tsx,jsx}": ["npm run lint-fix", "prettier --write"]
}
```

进行到这里，`git`提交的时候，会自动根据项目`ESLint`和`prettier`规范修复代码并提交，如果遇到无法自动修复的，会取消提交

#### 9. （可选）配置.eslintignore 和 .prettierignore

---

#### Caution：

- npx lint-staged 报错见常见问题Q3