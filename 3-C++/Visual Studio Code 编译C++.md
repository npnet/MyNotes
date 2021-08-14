## Visual Stdio Code 编译C/C++

### 1.下载VSCode

[VSCode下载地址](https://code.visualstudio.com/Download)



### 2.安装VSCode插件

1. C/C++
2. Code Runner



### 3.安装MINGW-W64

[Mingw-w64下载地址](http://mingw-w64.org/doku.php/download)

安装页面配置：

![Mingw-w64安装页面.jpg](https://i.loli.net/2021/08/13/U5JbIvTwKxNV6Cr.jpg)



### 4.配置调试环境

1. C++  .vscode配置

   创建.vscode文件夹及 launch.json 和 tasks.json 文件，工程路径下不能有中文
   <kbd>launch.json</kbd>

   ```json
   {
       "version": "0.2.0",
       "configurations": [
           {
               "name": "C++ Launch (GDB)",                
               "type": "cppdbg",                         
               "request": "launch",                        
               "targetArchitecture": "x86",                
               "program": "${workspaceRoot}\\${fileBasenameNoExtension}.exe",                 //fileBasenameNoExtension
               "miDebuggerPath":"D:\\Software\\MinGw-w64\\mingw64\\bin\\gdb.exe", 
               "args": [],     
               "stopAtEntry": false,                  
               "cwd": "${workspaceRoot}",                  
               "externalConsole": true,                  
               "preLaunchTask": "g++"　　                  
               }
       ]
    }
   ```

   <kbd>tasks.json</kbd>

   ```json
   {
       "version": "2.0.0",
       "command": "g++",
       "args": ["-g","-std=c++11","${file}","-o","${workspaceRoot}\\${fileBasenameNoExtension}.exe"],
       "problemMatcher": {
           "owner": "cpp",
           "fileLocation": ["relative", "${workspaceRoot}"],
           "pattern": {
               "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
               "file": 1,
               "line": 2,
               "column": 3,
               "severity": 4,
               "message": 5
           }
       },
       "tasks": [
           {
               "label": "g++",
               "command": "g++",
               "args": [
                   "-g",
                   "-std=c++11",
                   "${file}",
                   "-o",
                   "${workspaceRoot}\\${fileBasenameNoExtension}.exe"
               ],
               "problemMatcher": {
                   "owner": "cpp",
                   "fileLocation": [
                       "relative",
                       "${workspaceRoot}"
                   ],
                   "pattern": {
                       "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                       "file": 1,
                       "line": 2,
                       "column": 3,
                       "severity": 4,
                       "message": 5
                   }
               },
               "group": {
                   "_id": "build",
                   "isDefault": false
               }
           }
       ]
   }
   ```

2. C  .vscode配置

   <kbd>launch.json</kbd>

   ```json
   {
       "version": "0.2.0",
       "configurations": [
           {
               "name": "C/C++",
               "type": "cppdbg",
               "request": "launch",
               "program": "${fileDirname}/${fileBasenameNoExtension}.exe",
               "args": [],
               "stopAtEntry": false,
               "cwd": "${workspaceFolder}",
               "environment": [],
               "externalConsole": true,　　//弹出黑框使用true，不弹出使用false
               "MIMode": "gdb",
               "miDebuggerPath": "D:/Software/MinGw-w64/mingw64/bin/gdb.exe",　　//选择gbd.exe的绝对路径
               "preLaunchTask": "compile",
               "setupCommands": [
                   {
                       "description": "Enable pretty-printing for gdb",
                       "text": "-enable-pretty-printing",
                       "ignoreFailures": true
                   }
               ],
           },
       ]
   }
   ```

   <kbd>tasks.json</kbd>

   ```json
   {
       "version": "2.0.0",
       "tasks": [
           {
               "type": "shell",
               "label": "compile",
               "command": "gcc",　　//c文件就用gcc，cpp文件就用g++
               "args": [
                   "-g",
                   "${file}",
                   "-o",
                   "${fileDirname}\\${fileBasenameNoExtension}.exe"
               ],
               "problemMatcher": [
                   "$gcc"
               ],
               "group": {
                   "kind": "build",
                   "isDefault": true
               }
           }
       ]
   }
   ```

   

### 5.调试

完成代码编写后，<kbd>Ctrl</kbd> + <kbd>F5</kbd> 进行调试