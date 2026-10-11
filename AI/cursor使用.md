### 1. 插件安装

Ctrl+Shift+X

#### Extension Pack for Java

| 插件                          | 主要功能                                                     | 一句话点评                         |
| :---------------------------- | :----------------------------------------------------------- | :--------------------------------- |
| **Extension Pack for Java**   | Java开发全家桶，含语言支持、调试、Maven/Gradle等核心工具     | 在Cursor中开发Java的**必备基础包** |
| **MybatisX**                  | MyBatis框架增强插件，支持Mapper接口与XML的相互跳转、代码生成 | MyBatis用户的**效率神器**          |
| **IntelliJ IDEA Keybindings** | 将Cursor的快捷键映射为IntelliJ IDEA风格                      | **从IDEA迁移用户的福音**           |
| **XML Tools**                 | 提供XML的格式化、XPath查询、树形视图浏览等功能               | 处理XML文档的瑞士军刀              |
| **Prettier - Code formatter** | 主流的代码格式化工具，可一键美化多种语言代码，保持风格统一   | ** 终结代码风格争论**的利器        |
|                               |                                                              |                                    |







### 2. 全局配置



**用户级别配置**

打开命令面板

输入**Preferences: Open User Settings (JSON)**

```json
{
  "workbench.settings.applyToAllProfiles": [
      "editor.fontFamily",
      "editor.fontSize"
  ],
  "workbench.colorTheme": "Default Dark Modern",
  "workbench.iconTheme": "material-icon-theme",
  "material-icon-theme.hidesExplorerArrows": false,
  "workbench.tree.indent": 8,
  "workbench.tree.renderIndentGuides": "none",
  "workbench.sideBar.location": "left",
  "workbench.statusBar.visible": true,
  "workbench.editor.showTabs": "multiple",
  "workbench.layoutControl.enabled": false,
  "workbench.startupEditor": "none",
  "workbench.tips.enabled": false,
  "editor.minimap.enabled": false,
  "editor.guides.indentation": false,
  "editor.renderWhitespace": "none",
  "editor.renderLineHighlight": "none",
  "editor.matchBrackets": "never",
  "editor.lightbulb.enabled": "off",
  "editor.hover.enabled": false,
  "editor.showFoldingControls": "never",
  "editor.scrollbar.horizontal": "auto",
  "editor.scrollbar.vertical": "auto",
  "editor.overviewRulerBorder": false,
  "editor.cursorBlinking": "solid",
  "editor.cursorSmoothCaretAnimation": "on",
  "editor.tabSize": 4,
  "editor.detectIndentation": false,
  "editor.fontFamily": "Menlo, Monaco, 'Courier New', monospace,'FiraCode Nerd Font'",
  "editor.fontSize": 15,
  "editor.fontLigatures": true,
  "editor.lineHeight": 1.5,
  "breadcrumbs.enabled": false,
  "explorer.autoReveal": true,
  "explorer.confirmDragAndDrop": false,
  "explorer.confirmDelete": true,
  "explorer.compactFolders": false,
  "git.decorations.enabled": false,
  "scm.diffDecorations": "none",
  "extensions.ignoreRecommendations": true,
  //自动保存:延迟
  "files.autoSave": "afterDelay",
  //自动保存延迟时间:1000 毫秒
  "files.autoSaveDelay": 1000,
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true,
  "terminal.integrated.fontFamily": "Dank Mono, JetBrainsMono Nerd Font",
  "terminal.integrated.lineHeight": 1.5,
  "terminal.integrated.fontSize": 15,
  "terminal.integrated.env.osx": {
    "FIG_NEW_SESSION": "1"
  },
  // "window.titleBarStyle": "native",
  "workbench.editor.enablePreview": false,
  "background.fullscreen": {
    // Local images can be dragged into the browser to quickly get the file protocol address from the address bar
    "images": [],
    "opacity": 0.6,
    "size": "cover",
    "position": "center",
    "interval": 0,
    "random": false
  },
"background.enabled": true,
"git.autofetch": true,
}
```

Java配置, 如果想要把java也配置在整个用户级别，可以把以下配置粘贴上去

```json
{
    //启动窗口、打开文件夹、保存文件时的自动编译开关
    //影响启动速度，如有需要可启动后再手动打开
    "java.autobuild.enabled": false,
    //debug 启动时自动编译:关闭
    //如果打开，则应用启动前需要编译整个项目，耗时 1.5-5 分钟
    //建议手工编译，可提升启动速度
    "java.debug.settings.forceBuildBeforeLaunch": false,
    //debug 自动加载修改后的类
    "java.debug.settings.hotCodeReplace": "auto",
    //保存时自动编译:开启
    //但似乎此参数无效，实操经验是：
    //倘若 java.autobuild.enabled 为 true ，则保存后自动编译
    //倘若 java.autobuild.enabled 为 false ，则保存后不自动编译
    "java.compile.onSave":true,
    //问题装饰:关闭
    "problems.decorations.enabled": false,
 
    //未使用导入:忽略
    "editor.unusedImports.severity": "ignore",
    //未使用变量:隐藏
    "editor.showUnused": false,

    "java.jdt.ls.vmargs": "-XX:+UseParallelGC -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -Dsun.zip.disableMemoryMapping=true -Xmx10G -Xms2G -Xlog:disable", //设置内存大小
    "java.dependency.packagePresentation": "hierarchical",  //JAVA 项目层级展示
    "java.configuration.updateBuildConfiguration": "automatic",
    "java.compile.nullAnalysis.mode": "disabled",           //null 分析:关闭
    "java.configuration.maven.globalSettings": "D:\\Software\\Java\\apache-maven-3.8.9\\settings.xml",
    "java.configuration.maven.userSettings": "D:\\Software\\Java\\apache-maven-3.8.9\\settings.xml",
    "java.maven.downloadSources": true,
    "java.sources.organizeImports.starThreshold": 99,
    "java.sources.organizeImports.staticStarThreshold": 99,
    "java.project.sourcePaths": [
        "src/main/java",
        "src/test/java"
    ],
    "java.project.outputPath": "target/classes",
    "java.project.referencedLibraries": [
        "target/dependency/*.jar",
        "lib/**/*.jar"
    ],
    "java.format.settings.url": "https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml",
    "java.saveActions.organizeImports": true,
    "java.server.launchMode": "Standard",
    "java.import.maven.enabled": true,
    "java.contentProvider.preferred": "fernflower",
    "java.codeGeneration.hashCodeEquals.useJava7Objects": true,
    "java.codeGeneration.generateComments": true,
    "java.sharedIndexes.enabled": "auto",
    "java.maxConcurrentBuilds": 1,
    "java.completion.enabled": true,
    "java.signatureHelp.enabled": true,

    // 增强Maven集成
    //maven 视图:分层
    "maven.view": "hierarchical",
    "maven.executable.path": "D:\\Software\\Java\\apache-maven-3.8.9\\bin\\mvn",
    "maven.settingsFile": "D:\\Software\\Java\\apache-maven-3.8.9\\settings.xml",
    "maven.showInExplorerContextMenu": true,

    // Java语言服务器设置
    "java.eclipse.downloadSources": true,
    "java.trace.server": "verbose",
    "java.debug.settings.enableRunDebugCodeLens": true,
    "java.debug.settings.showHex": true,
    "java.debug.settings.showStaticVariables": true,
    "java.debug.settings.showQualifiedNames": false,
    "java.debug.settings.maxStringLength": 0,
    "java.debug.settings.numericPrecision": 0,
    "java.debug.settings.onBuildFailureProceed": true,        //构建失败继续:开启
    "java.debug.settings.console": "integratedTerminal",

    // 代码导航和补全
    "java.symbols.includeSourceMethodDeclarations": true,
    "java.foldingRange.enabled": true
}
```

注意：

- "java.configuration.maven.globalSettings" 指向自己的 maven settings.xml 文件
- "java.configuration.maven.userSettings"  指向自己的 maven  settings.xml 文件
- "maven.executable.path"  指向 mvn 工具（如果mvn不在环境变量PATH中）
- "maven.settingsFile" 指向自己的 maven settings.xml 文件