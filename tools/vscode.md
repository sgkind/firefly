vscode设置
===

## 修改右侧预览框的大小
1. File->Preferences->setting
2. 搜索框中搜索minimap
3. 将Editor > Minimap: Max Column 修改为合适的值

### 跳转后在新标签中打开
1. File->Preferences->setting
2. 搜索框中搜索preview
3. 取消Editor->Rename: Enable Preview

### 添加垂直分割线
1. File->Preferences->setting
2. 搜索框中搜索Rulers
3. Editor: Rulers中的Edit in settings.json
4. 修改
```
"editor.rulers": [
    79
],
```

