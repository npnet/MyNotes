## 使用VSCode中的Code Runner插件执行golang代码
在`settings.json`配置文件中，添加Code Runner插件的go执行命令

```json
{
    "code-runner.executorMap": {
        "go": "cd $dir && go run .",
    }
}
```

