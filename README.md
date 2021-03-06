# Eslint
eslint规约
Eslint 代码规范整合记录

# 前景准备

咨询了阿里那边的人员，得知他们的一套规范目前是开发对外的，虽然在阿里云的代码扫描的文档里面只写了 Eslint默认规则集。所以我们拿到了https://github.com/alibaba/f2e-spec 这个链接。

文档写的很清晰明了，可以自行查阅。阿里甚至还封装了 F2ELint 这个 npm 包，它收敛屏蔽了这些依赖和配置细节，提供简单的 CLI 和 Node.js API，让项目能够一键接入、一键扫描、一键修复、一键升级，并为项目配置 git commit 卡口，降低项目接入规约的成本。

# 实际操作


## 安装

在终端执行：
```javascript
npm install f2elint -g
```
安装完成后，可执行 f2elint -h 以验证安装成功。（npm可能需要高版本）

## 使用

### f2elint init：一键接入

在项目根目录执行 f2elint init，即可一键接入规约，为项目安装规约 Lint 所需的依赖和配置。

```javascript
f2elint init
```

具体会做以下事情：

安装各种依赖：包括 Linter 依赖，如 ESLint、stylelint、commitlint、markdownlint 等；配置依赖，如 eslint-config-ali、stylelint-config-ali、commitlint-config-ali、markdownlint-config-ali 等写入各种配置文件，包括：
- .eslintrc.js、.eslintignore：ESLint 配置（继承 eslint-config-ali）及黑名单文件
- .stylelintrc.js、.stylelintignore：stylelint 配置（继承 stylelint-config-ali）及黑名单文件
- commitlint.config.js：commitlint 配置（继承 commitlint-config-ali）
- .markdownlint.json、.markdownlintignore：markdownlint 配置及黑名单文件
- .prettierrc.js：符合规约的 Prettier 配置
- .editorconfig：符合规约的 editorconfig
- .vscode/extensions.json：写入规约相关的 VSCode 插件推荐，包括 ESLint、stylelint、markdownlint、prettier 等
- .vscode/settings.json：写入规约相关的 VSCode 设置，设置 ESLint 和 stylelint 插件的 validate 及保存时自动运行 fix，如果选择使用 Prettier，会同时将 prettier-vscode 插件设置为各前端语言的 defaultFormatter，并配置保存时自动格式化
- f2elint.config.js：f2elint 包的一些配置，如启用的功能等
配置 git commit 卡口：使用 husky 设置代码提交卡口，在 git commit 时会运行 f2elint commit-file-scan 和 f2elint commit-msg-scan 分别对提交文件和提交信息进行规约检查。f2elint commit-file-scan 默认仅对 error 问题卡口，如果你想对 warn 问题也卡口，可以增加 --strict 参数以开启严格模式

> 注 1：如果项目已经配置过 ESLint、stylelint 等 Linter，执行 f2elint init 将会提示存在冲突的依赖和配置，并在得到确认后进行覆盖。

>  注 2：如果项目的 .vscode/ 目录被 .gitignore 忽略，可以在拉取项目后单独执行 f2elint init --vscode 命令写入 .vscode/extensions.json 和 .vscode/settings.json 配置文件


### f2elint scan：一键扫描

在项目的根目录执行命令，即可扫描项目的规约问题：



支持下列参数：


-q --quiet 仅报告 error 级别的问题
-o--output-report 输出扫描出的规约问题日志
-i--include <dirpath> 指定要进行规约扫描的目录
--no-ignore 忽略 eslint 的 ignore 配置文件和 ignore 规则

> 注 1：事实上，你可以在任意目录执行 f2elint scan，F2ELint 会根据文件类型、JSON 等特征嗅探项目类型。但我们还是推荐在执行过 f2elint init 的项目根目录执行 f2elint scan，以得到最准确的扫描结果。

> 注 2：F2ELint 会根据项目内有无 eslint 和 stylelint 配置文件判断使用项目的配置文件还是 F2ELint 默认配置进行扫描。若使用项目的，在未安装依赖时会帮其安装（执行 npm i）。若使用项目配置扫描失败，则使用默认配置扫描


### f2elint fix：一键修复

在项目的根目录执行命令，即可修复部分规约问题.

```javascript
f2elint fix
```

支持下列参数：
-i --include <dirpath> 指定要进行修复扫描的目录
--no-ignore 忽略 eslint 的 ignore 配置文件和 ignore 规则
注意请 review 下修复前后的代码，以免工具误修的情况。


### f2elint commit-file-scan 提交文件扫描

在 git commit 时对提交文件进行规约问题扫描，需配合 git 的 pre-commit 钩子使用。

支持下列参数：
-s --strict 严格模式，对 warn 和 error 问题都卡口，默认仅对 error 问题卡口

### f2elint commit-msg-scan 提交信息扫描

git commit 时对 commit message 的格式进行扫描（使用 commitlint），需配合 husky 的 commit-msg 钩子使用。


# 结果
一顿操作过后我们基本上算是已经修复了80%的问题了，但是剩下的20%因为需要人工干涉取舍才能完成修复。

- Expected '===' and instead saw '=='.    eslint(eqeqeq) 

  项目中最多的warning，目的是为了类型安全。虽然只是警告，修复起来也不算困难，但是项目中实在是太多太多太多了。。。

- Assignment to property of function parameter 'xxxx'  eslint(no-param-reassign)

  顾名思义就是不能给函数参数赋值类型。

  这个问题在项目中也很常见，其次也不是很好修复。
  
  规则的错误代码示例：
  ```javascript
    /*eslint no-param-reassign: "error"*/
    function foo(bar) {
      bar = 13;
    }
    function foo(bar) {
      bar++;
    }
  ```
  规则的正确代码示例：
  ```javascript
    /*eslint no-param-reassign: "error"*/
    function foo(bar) {
      var baz = bar;
    }
   ```
 - no-nested-ternary  不允许有三元表达式，目的好像是嵌套三元表达式会使代码更难理解。
  
 - eval can be harmful.eslint(no-eval）  
  
  JavaScript的eval()功能是潜在的危险，经常被滥用。eval()在不受信任的代码上使用可以打开一个程序，最多可以进行多种不同的注入攻击 eval()在大多数情况下的使用可以取代对问题更好的替代方法。
  
 - indent 首行缩进 
  
 - Unexpected var, use let or const instead.eslint(no-var)   
  
  除了非 ES6环境外，如果开始将 ES6引入其代码库的现有 JavaScript 项目可能不希望应用此规则，如果从var迁移到let的代价过高。
  
 - Missing radix parameter.eslint(radix)  
  
   第二个参数必填。如果您不想强制执基数值的存在或省略，则可以关闭此规则。
  
 - Unexpected string concatenation. eslint(prefer-template)  
  
   在 ES2015（ES6）中，我们可以使用模板文字而不是字符串连接。
  
 - Expected 1 empty line after import statement not followed by another import  import/newline-after-import  
  
   import语句后的空行后面没有另一个import。（导入/导入后换行）
  
 - The body of a for-in should be wrapped in an if statement to filter unwanted properties from the prototype.eslint(guard-for-in)
  
   循环遍历对象for in将包含通过原型链继承的属性。此行为可能会导致 for 循环中出现意外的项目。
  
   正确示例：
   ```javascript
      /*eslint guard-for-in: "error"*/
      for (key in foo) {
        if (Object.prototype.hasOwnProperty.call(foo, key)) {
          doSomething(key);
        }
        if ({}.hasOwnProperty.call(foo, key)) {
          doSomething(key);
        }
      }
    ```
  - no-lonely-if 
    如果一个if陈述是该else块中唯一的陈述，那么使用一个else if表格通常会更清晰。
    ```javascript
      if (foo) {
         // ...
      } else {
        if (bar) {
          // ...
        }
      }
    ```
    应改写为：
    ```javascript
      if (foo) {
        // ...
      } else if (bar) {
        // ...
      }
    ```
  - xxxx is better written in dot notation.eslintdot-notation eslint(dot-notation)
      
      在 JavaScript 中，可以使用点符号（foo.bar）或方括号表示法（foo["bar"]）来访问属性。但是，点符号通常是首选，因为它更易于阅读，不会太冗长，而且在侵略性 JavaScript 最小化器中效果更好。
  
  - Expected longform method syntax for string literal keys.eslint(object-shorthand)
  
    ECMAScript 6为定义对象字面值方法和属性提供了一个简明的形式。这种语法可以使定义复杂的对象文字更清晰。
  
    ```javascirpt
       var foo = {
         w: function() {},
         x: function *() {},
         [y]: function() {},
         z: z
       };
    ```
    应改写为：
      
    ```javascript
       var foo = {
           w() {},
           *x() {},
           [y]() {},
            z
       };
    ```
    
   















