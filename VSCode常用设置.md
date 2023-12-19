# VSCode常用设置

### 1. 配置Vue代码片段

1. shift+ctrl+p 配置用户代码片段

2. vue.json

3. 编辑模板代码

   ```json
   {
   	"Print to console": {
   		"prefix": "vue2",
   		"body": [
   			"<template>",
   			"  <div>",
   			"  </div>",
   			"</template>",
   			"<script>",
   			"export default {",
   			"	name: '$TM_FILENAME_BASE',",
   			"  	data () {",
   			"    	return {",
   			" 		}",
   			"	},",
   			"}",
   			"</script>",
   			"<style lang=\"scss\">",
   			"</style>",
   			"$2"
   		],
   	}
   }
   ```

### 2. 自定义eol

1. 文件 - 首选项 - 设置
2. eol