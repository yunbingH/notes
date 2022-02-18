vs code

import 同一个项目下的本地package，报can not find in any GOROOT or GOPATH.

```
第 1 步：打开 Vscode，然后转到设置。

第 2 步：在搜索栏中，输入 gopls

第 3 步：就在它的下方，您会找到 settings.json，点击它

第4步：粘贴下面的代码他们的
"gopls": {"experimentalWorkspaceModule":true}

第 5 步：保存并重新启动 Vscode，您现在可以开始了。
```

