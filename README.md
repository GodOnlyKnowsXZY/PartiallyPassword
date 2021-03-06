# Typecho 文章部分加密插件（PartiallyPassword）

Typecho 文章部分加密插件（PartiallyPassword）支持对某一篇文章的特定部分创建密码，访客需要正确输入密码才能查看内容。

## 安装 Installation

### A. 直接下载

访问本项目的 Release 页面，下载最新的 Release 版本，解压后将其中的文件夹重命名为 `PartiallyPassword`（如果它原本不是这个名字）并移动到 Typecho 插件目录下。

### B. 从仓库克隆 

在 Typecho 插件目录下启动终端，执行命令即可。

```bash
git clone -b master --single-branch https://github.com/wuxianucw/PartiallyPassword.git
```

或下载压缩包（`Download ZIP`）并解压，将其中的文件夹重命名为 `PartiallyPassword` （如果它原本不是这个名字）并移动到 Typecho 插件目录下。

## 使用方法 Usage

### 初始化

#### 全新安装

启用插件，即完成全部初始化工作。默认配置是一套极简风格的密码输入框，你也可以根据主题特性进行自定义修改。

#### 从旧版本升级

首先备份插件配置（手动将其拷贝到文本文档中等方法），然后禁用旧版本插件。用新版本插件文件覆盖旧版本插件文件，再启用新版本并从备份中恢复配置。

**注意：** 如果旧版本是 v2.x 版本，请在删除插件目录下的 `upgrade.lock` 文件后运行一次 `Upgrade.php`（在浏览器中打开 `http(s)://你的博客地址/usr/plugins/PartiallyPassword/Upgrade.php`），它将会自动升级 v2.x 的文章自定义字段配置到 v3 配置（JSON 方案）。

**确认无误后** ，删除 `Upgrade.php`，保留脚本自动创建的 `upgrade.lock` 以防止误操作，并将 `upgrade.log` 拷贝到其他位置备用。如果在执行脚本中遇到问题（`Error`、`Exception` 字样），请提出 issue（类型为 `Bug report`）并附带上 `upgrade.log`。

### 调用方法举例

#### 基础用法

**在书写加密语法之前，请先将对应文章下方“自定义字段”中“是否开启文章部分加密”一项调整为“开启”状态。** 该项目默认为“关闭”状态，在此情况下，任何加密语法都不会被解析。

下面的所有例子都包含一个“文本部分”和一个“配置部分”，其中上方的“文本部分”是需要在 Typecho 编辑器中书写的内容，下方的“配置部分”是需要在文章下方“自定义字段”中“密码组”一项内填入的内容。

```text
不需要密码的内容

[ppblock]
输入密码可见的内容
[/ppblock]

不需要密码的内容
```

```json
["123456"]
```

这就是一个最简单的例子。你也可以进一步给加密块添加附加信息：

```text
[ppblock ex="请输入密码"]
输入密码可见的内容
[/ppblock]
```

```json
["123456"]
```

附加信息将会在输入密码处显示。

如果你想书写一段 `[ppblock]...[/ppblock]` 形式的文本，但不希望它被解析，请使用 `[[ppblock]...[/ppblock]]`，两侧多余的方括号会被自动移除。

#### 插入多个加密块

```text
不需要密码的内容

[ppblock]
需要密码的内容 A，id = 0
[/ppblock]

不需要密码的内容

[ppblock pwd="喵"]
需要密码的内容 B，id = 1
[/ppblock]

不需要密码的内容

[ppblock ex="给我密码"]
需要密码的内容 C，id = 2
[/ppblock]

不需要密码的内容
```

```json
{
    "0": "123456",
    "喵": "miao~",
    "2": "000000"
}
```

每个加密块都会被赋予一个 `id`，它从 0 开始依次增加。加密块密码的寻找逻辑如下：

1. 检查 `pwd` 属性
    - 若该属性存在，从第 2 步继续后续操作 →
    - 若该属性不存在，从第 3 步继续后续操作 →
2. 寻找 JSON 中是否有索引为 `pwd` 的值的项目
    - 是，使用该项目作为当前块的密码，结束 √
    - 否，从第 4 步继续后续操作 →
3. 寻找 JSON 中是否有索引为 `id` 的值的项目
    - 是，使用该项目作为当前块的密码，结束 √
    - 否，从第 4 步继续后续操作 →
4. 寻找 JSON 中是否有索引为 `fallback` 的项目
    - 是，使用该项目作为当前块的密码，结束 √
    - 否，当前块展现为“密码未设置”错误提示 ×

在上例中，三个加密块的密码依次是 `123456`、`miao~` 和 `000000`。

下面的例子演示 `fallback` 的功能：

```text
不需要密码的内容

[ppblock]
需要密码的内容 A，id = 0
[/ppblock]

不需要密码的内容

[ppblock pwd="喵" ex="给我密码"]
需要密码的内容 B，id = 1
[/ppblock]

不需要密码的内容
```

```json
{
    "1": "miao~",
    "fallback": "123456"
}
```

这个例子中，两个加密块的密码都是 `123456`。第二个加密块虽然指定了 `pwd` 属性，但密码组中没有对应的项目，因此也不再检查是否有符合 `id` 的项目，而是直接使用 `fallback`。

这种使用 `pwd` 属性指定密码的方式称为“命名密码”。当只使用索引时，密码组配置可以简写为一个 `string[]` 类型的 JSON 数组，例如：

```text
不需要密码的内容

[ppblock]
需要密码的内容 A，id = 0
[/ppblock]

不需要密码的内容

[ppblock ex="给我密码"]
需要密码的内容 B，id = 1
[/ppblock]

不需要密码的内容
```

```json
["123456", "miao~"]
```

这在加密块比较少或密码不重用时非常方便。

#### 不同密码对应不同内容

自 v3.0.0 起，引入了一种新的标记 `ppswitch`。下面是一个例子：

```text
不需要密码的内容

[ppswitch]
公共内容（可选）
[case pwd="p1"]
输入了 p1 的情况
[/case]
[case pwd="p2"]
输入了 p2 的情况
[/case]
[/ppswitch]
```

```json
{
    "p1": "111111",
    "p2": "222222"
}
```

当未输入密码时，展现为：

```text
不需要密码的内容

```

如果输入 `111111`（即 `p1` 的值），则展现为：

```text
不需要密码的内容

公共内容（可选）
输入了 p1 的情况
```

输入 `222222`（即 `p2` 的值）后的展现类推。

有两点需要特别注意：

1. `case` 标记必须指定 `pwd` 属性，否则无论如何都不会显示，而且，除非指定 `pwd="fallback"`，否则不会在找不到密码时自动采用 `fallback` 的值；
2. `ppswitch` 与 `ppblock` 共用一套 `id` 系统，尽管 `ppswitch` 默认不会寻找密码组中索引为 `id` 的值的项目。

关于第二点，下面这个例子可能能够帮助理解：

```text
[ppblock]
这个加密块的 id = 0
[/ppblock]

[ppswitch]
这个加密块的 id = 1
[case pwd="0"]
你输入了上一个加密块的密码
[/case]
[case pwd="2"]
你输入了下一个加密块的密码
[/case]
[/ppswitch]

[ppblock]
这个加密块的 id = 2
[/ppblock]
```

```json
{
    "0": "000000",
    "2": "123456"
}
```

可以看到，中间的 `ppswitch` 占用了一个 `id`，但是无法直接通过它的 `id` 为它指定密码（除非设置 `[case pwd="1"]`，但这时它是一个索引为 `1` 的命名密码）。这种设计是出于多个 `case` 时难以规定默认行为的考虑。

### 提示

- 请勿不成对或嵌套地使用标记，它的展现无法预期。

## TODO List

- [x] 在 `Widget_Abstract_Contents` 的 `excerpt` 下挂接函数，屏蔽所有 `[ppblock]` 以及其中的内容，不判断 Cookie。（Since v1.1.0）
- [x] ~~寻找一个方案可以直接操作 `$widget->text` 取出的内容，实现完美屏蔽。~~ 已经更改为在 `Widget_Abstract_Contents` 的 `filter` 下挂接插件实现方法，这样操作后从 Widget 中取出的数据已经全部进行了过滤，除非直接读取数据库，否则理论上不存在加密区块不解析的情形。（Since v2.0.0）
- [x] ~~现有的鉴权逻辑较为不完善，应增加提交密码时的后端相关处理，并合理优化流程。~~ 已经完全交由后端处理 Cookie，流程变更为直接向文章页面 POST 数据。（Since v2.0.0）
- [x] ~~默认外观需要优化，包括样式和插入位置。~~ 已经完成优化，现在的默认样式是一套极简风格的密码输入框。（Since v1.1.1）公共 HTML 的插入位置变更为页头和页脚。（Since v2.0.0）
- [x] ~~考虑增加加密区块语法支持，后续将可能支持更加复杂的语法。具体方案暂时未定。~~ 新增 `ppswitch` 语法，能够实现不同密码对应不同内容（[#2](https://github.com/wuxianucw/PartiallyPassword/issues/2)）。（Since v3.0.0）
