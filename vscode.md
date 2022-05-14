# Config

## Shortcut Keys

- 반복되는 단어 한방에 수정 : Ctrl + D
- 클릭하는 곳마다 커서 생성 : Alt + Click
- 코드 위/아래로 움직이기 : Alt + ↑ / ↓
- 코드 복사해서 위/아래로 움직이기 : Alt + Shift + ↑ / ↓
- 코드 블록 코멘트 처리하기 : Ctrl + /
- 선택된 영역에 커서 만들기 : Alt + Shift + i 
- 마우스가 가는 곳 마다 커서 만들기 : Alt + Shift + Mouse Drag
- 파일 맨 위-아래로 한번에 이동하기 : Ctrl + Home / End
- 사이드바 숨기기 : Ctrl + B



## .vscode/settings.json


```json
{
    // vs code
    "extensions.ignoreRecommendations": true,
    "telemetry.telemetryLevel": "off",
    "window.titleBarStyle": "custom",
    "workbench.editor.historyBasedLanguageDetection": true,
    "workbench.iconTheme": "vscode-icons",
    "workbench.settings.useSplitJSON": true,
    "workbench.startupEditor": "newUntitledFile",

    // editor
    "breadcrumbs.enabled": true,
    "diffEditor.ignoreTrimWhitespace": false,
    "editor.fontFamily": "'Ubuntu Mono', 'Bitstream Vera Sans Mono', 'Dejavu Sans Mono', 'Consolas', 'Droid Sans Mono', 'monospace', monospace, 'Droid Sans Fallback'",
    "editor.fontSize": 14,
    "editor.minimap.enabled": false,
    "editor.renderWhitespace": "none",
    "editor.suggestSelection": "first",
    "explorer.confirmDelete": false,
    "explorer.confirmDragAndDrop": false,
    "terminal.integrated.fontSize": 14,
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",

    // pasteImage
    "pasteImage.basePath": "${currentFileDir}/images",
    "pasteImage.path": "${currentFileDir}/images",
    "pasteImage.prefix": "./images/",
    
    // git
    "git.autofetch": true,
    "git.confirmSync": false,
    "scm.alwaysShowActions": true,
    "scm.alwaysShowRepositories": true,

    // java
    "java.jdt.ls.java.home": "/usr/local/opt/openjdk@11",
    "java.jdt.ls.vmargs": "-XX:+UseParallelGC -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -Dsun.zip.disableMemoryMapping=true -Xmx1G -Xms100m -javaagent:\"/Users/choegeun/.vscode/extensions/gabrielbb.vscode-lombok-1.0.1/server/lombok.jar\"",
    "java.project.importOnFirstTimeStartup": "automatic",
    "redhat.telemetry.enabled": false,

    // python
    "python.analysis.extraPaths": [".direnv/site-packages"],
    "workbench.editorAssociations": {
        "*.ipynb": "jupyter-notebook",
        "*.class": "default"
    },
    "[python]": {
        "editor.defaultFormatter": "ms-python.python"
    },
    "notebook.cellToolbarLocation": {
        "default": "right",
        "jupyter-notebook": "left"
    },
    "security.workspace.trust.untrustedFiles": "open",
    "python.formatting.autopep8Args": ["--max-line-length", "120", "--experimental"],

    // markdown

    // k8s
    "vs-kubernetes": {
        "vscode-kubernetes.helm-path.mac": "/Users/choegeun/.vs-kubernetes/tools/helm/darwin-amd64/helm",
        "vscode-kubernetes.helm-path.linux": "/home/dgdsingen/.vs-kubernetes/tools/helm/linux-amd64/helm",
        "vscode-kubernetes.minikube-path.linux": "/home/dgdsingen/.vs-kubernetes/tools/minikube/linux-amd64/minikube"
    },
    
    // etc
    "hediet.vscode-drawio.theme": "atlas",
    "vsicons.dontShowNewVersionMessage": true,
}
```



## keybindings.json


```json
[
    {
        "key": "ctrl+shift+j",
        "command": "editor.action.joinLines"
    },
    {
        "key": "ctrl+e",
        "command": "workbench.action.editor.changeEncoding"
    },
    {
        "key": "ctrl+space",
        "command": "editor.action.triggerSuggest",
        "when": "editorHasCompletionItemProvider && textInputFocus && !editorReadonly"
    },
    {
        "key": "ctrl+space",
        "command": "-editor.action.triggerSuggest",
        "when": "editorHasCompletionItemProvider && textInputFocus && !editorReadonly"
    }
]
```



## .vscode/extensions.json

```json
{
    "recommendations": [
        "alefragnani.Bookmarks",
        "christian-kohler.path-intellisense",
        "cweijan.vscode-mysql-client2",
        "eamodio.gitlens",
        "GabrielBB.vscode-lombok",
        "GitHub.vscode-pull-request-github",
        "googlecloudtools.cloudcode",
        "hediet.vscode-drawio",
        "humao.rest-client",
        "marp-team.marp-vscode",
        "mirone.milkdown",
        "ms-azuretools.vscode-docker",
        "MS-CEINTL.vscode-language-pack-ko",
        "ms-kubernetes-tools.vscode-kubernetes-tools",
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-toolsai.jupyter",
        "ms-toolsai.jupyter-keymap",
        "ms-toolsai.jupyter-renderers",
        "ms-vscode-remote.remote-containers",
        "ms-vscode-remote.remote-ssh",
        "ms-vscode-remote.remote-ssh-edit",
        "ms-vscode-remote.remote-wsl",
        "ms-vscode-remote.vscode-remote-extensionpack",
        "mushan.vscode-paste-image",
        "redhat.fabric8-analytics",
        "redhat.java",
        "redhat.vscode-commons",
        "redhat.vscode-xml",
        "redhat.vscode-yaml",
        "rintoj.blank-line-organizer",
        "shd101wyy.markdown-preview-enhanced",
        "Tyriar.sort-lines",
        "VisualStudioExptTeam.vscodeintellicode",
        "vscjava.vscode-java-debug",
        "vscjava.vscode-java-dependency",
        "vscjava.vscode-java-pack",
        "vscjava.vscode-java-test",
        "vscjava.vscode-maven",
        "vscode-icons-team.vscode-icons",
        "yzhang.markdown-all-in-one",
    ]
}
```


# Docker

## Remote Host 사용

settings.json에 아래와 같이 추가한다.

```json
"docker.host": "192.168.137.100:2375"
```

# Java


## {project-root}/.vscode/tasks.json


```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "shell",
            "label": "mvn compile",
            "command": "mvn -B compile",
            "group": "build"
        }
    ]
}
```



# Typescript


## {project-root}/.vscode/tasks.json


```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "typescript",
            "tsconfig": ".vscode/tsconfig.json", // root path = /
            "problemMatcher": [
                "$tsc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
        {
            "type": "shell",
            "label": "node t.js",
            "command": "node js/t.js",
            "group": "test"
        }
    ]
}
```



## {project-root}/.vscode/tsconfig.json


```json
{
    "compileOnSave": true,
    "compilerOptions": {
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "lib": [
            "es5",
            "dom"
        ],
        "module": "commonjs",
        "moduleResolution": "node",
        "noEmitOnError": true,
        "noImplicitAny": false,
        "outDir": "../js", // root path = /.vscode
        "removeComments": false,
        "sourceMap": true,
        "target": "es5",
        "watch": true
    },
    "include": [
        "../ts"
    ],
    "exclude": [
        "../js"
    ]
}
```



## Modules


### npm

예를 들어 node-fetch module을 사용하고 싶다면 아래와 같이 진행한다. 

1. node-fetch module을 설치한다. 


```sh
npm install node-fetch
```


2. node-fetch ts를 설치한다.


```sh
npm install --save @types/node-fetch
```


3. node-fetch module을 import해서 사용한다. 


```js
import * as nodeFetch from 'node-fetch';
nodeFetch.default('https://www.google.com').then(res => {
    res.text().then(text => {
        console.log(text);
    });
}).catch(res => {
    console.log(res);
});
```



### Local


```js
Test.ts
export class Test {
    id: string = 'default id';
    name: string = 'default name';
}

Main.ts
import { Test } from './Test';
var test = new Test();
console.log(test);
```

# VS Code Server

## Config

`curl -fsSL https://code-server.dev/install.sh | sh -s -- --dry-run`
로 먼저 설치 정보를 확인한다. 

`curl -fsSL https://code-server.dev/install.sh | sh`
로 설치한다. 

```sh
# run vs code server
code-server
```

보안상 기본적으로 localhost에만 expose 된다. 외부에 열기 위해서는 [nginx SSL & proxy_pass 설정](../platform/server.md)을 해준다.

# References
- [TypeScript Programming with Visual Studio Code](https://code.visualstudio.com/docs/languages/typescript) 
- [Tasks in Visual Studio Code](https://code.visualstudio.com/docs/editor/tasks#vscode) 
- [Developing inside a Container using Visual Studio Code Remote Development](https://code.visualstudio.com/docs/remote/containers) 
- [Advanced Container Configuration](https://code.visualstudio.com/docs/remote/containers-advanced) 
- [cdr/code-server: VS Code in the browser](https://github.com/cdr/code-server) 
- [headmelted/codebuilds: Community builds of Visual Studio Code for Chromebooks and Raspberry Pi](https://github.com/headmelted/codebuilds) 

