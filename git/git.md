### git 提交规范

```txt
注意：提交注释请按以下格式
[feat] xxxxxxx 代表提交的文件为新增功能，不需要cherry pick到其他分支
[fix] xxxxxx 代表提交的文件为修复bug，需要cherry pick到其他分支
其他格式类型可以参考(不做强制要求)：
[docs]: 只改动了文档相关的内容
[style]: 不影响代码含义的改动，例如去掉空格、改变缩进、增删分号
[sh]: 只改动了shell脚本相关的内容 
[sql]: 只改动了sql脚本相关的内容
[build]: 构造工具的或者外部依赖的改动，例如webpack，npm
[revert]: 执行git revert打印的message
```

