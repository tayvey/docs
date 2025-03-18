# vscode C#发布配置

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "清空Release目录",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}/src/webtest"
            },
            "command": "mkdir -p ./bin/Release && rm -rf ./bin/Release/*",
            // "command": "Remove-Item -Path .\\bin\\Release\\* -Recurse -Force",
            "problemMatcher": "$msCompile"
        },
        {
            "label": "清空Publish目录",
            "dependsOn": "清空Release目录",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}/src/webtest"
            },
            "command": "mkdir -p ./bin/Publish && rm -rf ./bin/Publish/*",
            // "command": "Remove-Item -Path .\\bin\\Publish\\* -Recurse -Force",
            "problemMatcher": "$msCompile"
        },
        {
            "label": "发布",
            "dependsOn": "清空Publish目录",
            "type": "process",
            "options": {
                "cwd": "${workspaceFolder}/src/webtest"
            },
            "command": "dotnet",
            "args": [
                "publish",
                "-p:GenerateFullPaths=true",
                "-clp:NoSummary",
                "-p:Platform=AnyCPU",
                "-p:SelfContained=false",
                "-p:PublishSingleFile=false",
                "-c",
                "Release",
                "-f",
                "net8.0",
                "-r",
                "linux-x64",
                "-o",
                "bin/Publish"
            ],
            "problemMatcher": "$msCompile"
        }
    ]
}
```

