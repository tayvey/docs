# vscode C#发布配置

```json
{
    "version": "2.0.0",
    "tasks": [
        // 创建解决方案
        {
            "label": "创建解决方案",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "dotnet new sln -n ${input:slnName}",
            "problemMatcher": "$msCompile"
        },
        // 创建项目
        {
            "label": "创建项目",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "dotnet new ${input:projectType} -n ${input:projectName} -f ${input:projectFramework}",
            "problemMatcher": "$msCompile"
        },
        // 添加项目到解决方案
        {
            "label": "添加项目到解决方案",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "dotnet sln ${input:slnName}.sln add ${input:projectName}/${input:projectName}.csproj",
            "problemMatcher": "$msCompile"
        },
        // 调试构建 Debug
        {
            "label": "清空Debug目录",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}/Project"
            },
            "command": "mkdir -p ./bin/Debug && rm -rf ./bin/Debug/*",
            "problemMatcher": "$msCompile"
        },
        {
            "label": "调试构建",
            "dependsOn": "清空Debug目录",
            "type": "process",
            "options": {
                "cwd": "${workspaceFolder}/Project"
            },
            "command": "dotnet",
            "args": [
                "build",
                "-p:GenerateFullPaths=true",
                "-clp:NoSummary",
                "-p:Platform=AnyCPU"
            ],
            "problemMatcher": "$msCompile"
        },
        // 发布 Publish
        {
            "label": "清空Release目录",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}/Project"
            },
            "command": "mkdir -p ./bin/Release && rm -rf ./bin/Release/*",
            "problemMatcher": "$msCompile"
        },
        {
            "label": "清空Publish目录",
            "dependsOn": "清空Release目录",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}/Project"
            },
            "command": "mkdir -p ./bin/Publish && rm -rf ./bin/Publish/*",
            "problemMatcher": "$msCompile"
        },
        {
            "label": "发布",
            "dependsOn": "清空Publish目录",
            "type": "process",
            "options": {
                "cwd": "${workspaceFolder}/Project"
            },
            "command": "dotnet",
            "args": [
                "publish",
                "-p:GenerateFullPaths=true",
                "-clp:NoSummary",
                "-p:Platform=AnyCPU",
                "-p:SelfContained=true",
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
    ],
    "inputs": [
        {
            "id": "slnName",
            "type": "promptString",
            "description": "解决方案名称"
        },
        {
            "id": "optionId",
            "type": "pickString",
            "description": "项目类型",
            "options": [
                {
                    "label": "控制台",
                    "value": "console"
                },
                {
                    "label": "ASP.NET Core 空",
                    "value": "web"
                },
                {
                    "label": "内库",
                    "value": "classlib"
                }
            ]
        },
        {
            "id": "projectName",
            "type": "promptString",
            "description": "项目名称"
        },
        {
            "id": "projectFramework",
            "type": "pickString",
            "description": "框架版本",
            "options": [
                "netstandard2.1",
                "net6.0",
                "net8.0",
            ]
        }
    ]
}
```