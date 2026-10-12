# note-format-for-importing-anki

Turn book highlights and reading notes into Anki-importable flashcards.

把读书笔记（划线句子 + 批注）批量转换成 Anki 可直接导入的 `result.txt`：每条划线一行、逗号分隔字段、自动去重合并，导入 Anki 即成卡片。

## 它做了什么

1. 递归扫描程序运行目录下所有 `*anki.txt` 文件（如 `百年孤独anki.txt`）；
2. 解析书名、作者、章节、划线句子和批注；
3. 相同句子自动去重合并（批注追加，章节保留首次出现）；
4. 在当前目录生成 `result.txt`，供 Anki 导入。

## 输入格式

每个 `*anki.txt`（文件名以 `anki.txt` 结尾）按以下格式书写：

```text
百年孤独
加西亚·马尔克斯

◆ 第一章
“多年以后……”这个开头太经典了
>> 多年以后，面对行刑队，奥雷里亚诺·布恩迪亚上校将会回想起父亲带他去见识冰块的那个遥远的下午。

◆ 第二章
>> 世界新生伊始，许多事物还没有名字，提到的时候尚需用手指指点点。
```

解析规则：

| 行 | 含义 |
|----|------|
| 第 1 行 | 书名 |
| 第 2 行 | 作者 |
| `◆` 开头 | 章节标题（作为其后划线的出处） |
| `>` 或 `>>` 开头 | 划线句子（去掉前缀和首尾空白，即卡片正面） |
| 其余行 | 批注，附加到之后遇到的第一条划线句子 |

## 输出格式

生成 `result.txt`，每行一条卡片，字段以英文逗号分隔：

```text
划线句子,批注,书名 章节,作者,书名
```

上例的输出：

```text
多年以后，面对行刑队，奥雷里亚诺·布恩迪亚上校将会回想起父亲带他去见识冰块的那个遥远的下午。,“多年以后……”这个开头太经典了,百年孤独 ◆ 第一章,加西亚·马尔克斯,百年孤独
```

| # | 内容 | 建议 Anki 字段 |
|---|------|----------------|
| 1 | 划线句子 | Front（正面） |
| 2 | 批注（同一句重复出现时多条合并，换行分隔） | Back（背面） |
| 3 | 书名 + 章节 | Source（出处） |
| 4 | 作者 | Author |
| 5 | 书名 | Book |

## 使用

1. 把一本或多本 `*anki.txt` 放到某个文件夹；
2. 在该文件夹下运行程序（exe，或 `dotnet run --project <项目路径>`）；
3. 同目录生成 `result.txt`；
4. 打开 Anki → 文件 → 导入（或直接把 `result.txt` 拖进牌组列表）。

### Anki 导入设置

- **笔记类型**：需要 5 个字段。可在 工具 → 管理笔记模板 中复制 Basic，再添加 Source / Author / Book 三个字段，5 列会按顺序自动对应；
- **字段分隔符**：Comma（默认）；
- 首行不是表头，不要勾选“第一行为字段名”；
- 想在卡片背面显示出处，模板的背面模板可以写成：

```html
{{FrontSide}}
<hr id=answer>
{{Back}}
<br><small style="color:#888">{{Source}} · {{Author}}</small>
```

## 构建

```bash
git clone https://github.com/liushengliuyun/note-format-for-importing-anki.git
cd note-format-for-importing-anki
dotnet publish -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -o publish
```

生成单文件 `publish/GenAnkiData.exe`（自带 .NET 运行时，目标机器无需安装 .NET）。把它放到笔记文件所在目录运行即可；也可以不开单文件打包，直接 `dotnet run`（同样以运行时所在目录为扫描根）。Visual Studio 用户：打开 `GenAnkiData.csproj` → F5。

## 注意事项

- 文件请以 UTF-8 保存，否则中文乱码；
- 字段以英文逗号分隔，划线句子里若含英文逗号会被 Anki 拆列；介意的话把 `Program.cs` 中的分隔符改为 `\t` 并在 Anki 导入时选 Tab；
- 批注与划线的对应关系是“批注在上、划线在下”，写笔记时注意顺序。
