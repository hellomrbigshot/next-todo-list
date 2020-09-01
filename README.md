# next-todo-list
### 介绍
使用 vue3.0 实现的 TodoList。注意点有两个

1. todos 初始化使用 ```ref([])``` 而不是 ```reactive([])```
2. watch 中监控的 todos 可以直接保存，不用取 ```todos.value```

在 ```public/index.html``` 中使用了 ```cdn``` 样式

## 安装依赖
```
npm install
```

### 本地运行
```
npm run serve
```

### 打包
```
npm run build
```

### 在线预览
[点我预览](https://hellomrbigshot.github.io/next-todo-list/)
