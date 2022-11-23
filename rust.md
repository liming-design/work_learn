
## 环境搭建
### windows
```bash
下载 rustup-init.exe
安装
cargo new greeting 
cd ./greeting 
cargo build 
cargo run 

在windows安装rust后运行第一个项目报错linker link.exe not found
只需要执行2个命令
rustup toolchain install stable-x86_64-pc-windows-gnu
rustup default stable-x86_64-pc-windows-gnu
```

tasks.json
```
{
	"version": "2.0.0",
	"tasks": [
		{
			"label": "build",
			"type": "shell",
			"command": "cargo",
			"args": [
				"build"
			]
		}
	]
}
```
launch.json
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "preLaunchTask":"build",
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${workspaceFolder}/target/debug/${workspaceFolderBasename}.exe", 
            "args": [],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```