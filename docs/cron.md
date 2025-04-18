Cron 速查表
===

[Cron](https://en.wikipedia.org/wiki/Cron) 最适合安排重复性任务。 可以使用关联的 at 实用程序来完成一次性任务的调度。

Crontab 格式
------
<!--rehype:body-class=cols-2-->

### 格式

```
Min  Hour Day  Mon  Weekday
分钟  小时  天   月   周
```

-------

```bash
*    *    *    *    *   <要执行的命令>
┬    ┬    ┬    ┬    ┬
│    │    │    │    └─  星期几         (0=周日 .. 6=星期六)
│    │    │    └──────  月            (1..12)
│    │    └───────────  月份中的某天    (1..31)
│    └────────────────  小时           (0..23)
└─────────────────────  分钟           (0..59)
```

-------

| 字段          | 范围   | 特殊字符             |
|--------------|--------|--------------------|
| 分钟 Minute   | 0 - 59 | <kbd>,</kbd> <kbd>-</kbd> <kbd>*</kbd> <kbd>/</kbd>
| 小时 Hour     | 0 - 23 | <kbd>,</kbd> <kbd>-</kbd> <kbd>*</kbd> <kbd>/</kbd>
| 月份中的某天   | 1 - 31 | <kbd>,</kbd> <kbd>-</kbd> <kbd>*</kbd> <kbd>?</kbd> <kbd>/</kbd> <kbd>L</kbd> <kbd>W</kbd>
| 月 Month     | 1 - 12 | <kbd>,</kbd> <kbd>-</kbd> <kbd>*</kbd> <kbd>/</kbd>
| 星期几        | 0 - 6  | <kbd>,</kbd> <kbd>-</kbd> <kbd>*</kbd> <kbd>?</kbd> <kbd>/</kbd> <kbd>L</kbd> <kbd>#</kbd>
| 年 Year       | 1970–2099  | <kbd>,</kbd> <kbd>-</kbd>
<!--rehype:className=show-header-->

### 示例

| Example        | Description            |
|----------------|------------------------|
| `*/15 * * * *` | 每 15 分钟   |
| `0 * * * *`    | 每隔一小时   |
| `0 */2 * * *`  | 每 2 小时   |
| `15 2 * * *`   | 每天凌晨 2 点 15 分   |
| `15 2 * * ?`   | 每天凌晨 2 点 15 分   |
| `10 9 * * 5`   | 每周五上午 9:10   |
| `0 0 * * 0`    | 每个星期日的午夜   |
| `15 2 * * 1L`  | 每月最后一个星期一凌晨 2 点 15 分   |
| `15 0 * * 4#2` | 每个月的第二个星期四早上 00:15   |
| `0 0 0 1 * *`  | 每个月的 1 日(每月)   |
| `0 0 0 1 1 *`  | 每年 1 月 1 日(每年)   |
| `@reboot`      | 每次重启 _(非标准)_   |

### 特殊字符串

| 特殊字符串       | 意义                                            |
|----------------|----------------------------------------------------|
| @reboot        | 运行一次，在系统启动时 _(非标准)_ |
| @yearly        | 每年运行一次，“0 0 1 1 *” _(非标准)_ |
| @annually      | (与@yearly 相同)_(非标准)_ |
| @monthly       | 每月运行一次，“0 0 1 \* \*” _(非标准)_ |
| @weekly        | 每周运行一次，“0 0 \* \* 0” _(非标准)_ |
| @daily         | 每天运行一次，“0 0 \* \* \*” _(非标准)_ |
| @midnight      | (与@daily 相同)_(非标准)_ |
| @hourly        | 每小时运行一次，“0 \* \* \* \*” _(非标准)_ |
<!--rehype:className=show-header -->

### Crontab 命令

| -            | -                                           |
|--------------|---------------------------------------------|
| `crontab -e` | 如果不存在，则编辑或创建一个 crontab 文件       |
| `crontab -l` | 显示 crontab 文件 |
| `crontab -r` | 删除 crontab 文件 |
| `crontab -v` | 显示您上次编辑 crontab 文件的时间 _(非标准)_ |

轻松添加任务

```bash
echo "@reboot echo hi" \| crontab
```

### 特殊字符
<!--rehype:wrap-class=col-span-2-->

| 特殊字符             | 说明 |
|---------------------|------------|
`星号(*)`  | 匹配字段中的所有值或任何可能的值。
`横杆(-)`  | 用于定义范围。例如：第 5 个字段(星期几)中的 1-5 每个工作日，即星期一到星期五
`斜线 (/)` | 第一个字段(分钟)/15 表示每十五分钟或范围的增量。
`逗号(,)`  | 用于分隔项目。例如：第二个字段(小时)中的 2、6、8 在凌晨 2 点、早上 6 点和早上 8 点执行
`L`       | 仅允许用于 `月份中的某天` 或 `星期几` 字段，`星期几` 中的 `2L` 表示每个月的最后一个星期二
`井号 (#)` | 仅允许用于 `星期几` 字段，后面必须在 1 到 5 的范围内。例如，`4#1` 表示给定月份的“第一个星期四”。
`问号(?)`  | 可以代替“*”并允许用于月份和星期几。使用仅限于 cron 表达式中的 `月份中的某天` 或 `星期几`。
<!--rehype:className=show-header auto-wrap-->

另见
----

- [Devhints](https://devhints.io/cron) _(devhints.io)_
- [Crontab Generator](https://crontab-generator.org/) _(crontab-generator.org)_
- [Crontab guru](https://crontab.guru/) _(crontab.guru)_
