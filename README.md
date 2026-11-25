# 跨区用工查询的授权执行与结果续读

各记录目录保存独立的合成业务资料。SQL 文件仅包含原始关系表与已有行。manifest.json 记录库角色、表、行数及运行参数。timeline.json 记录后续客户端行为、来源提交或故障，用 pos 表示可比较的到达次序。同一 pos 按数组顺序。提交位置由相应记录的业务字段指定。

## 接入协议

主体授权由 grants 的 actor、purpose、district、有效区间及 revision 定义。may_columns 的值列出受托方可读取的附加列，公开的行标识、岗位标题、地区、企业编号始终在行授权内可读。企业的私有电话字段需要逐区授权。shared_codes 的复合身份为 district 和 code。`redaction=permit` 表示当前请求可接受直接输出列被省略，记录字段上附有列省略标记。无论是否允许省略，用于关联、过滤、排序和聚合的列仍需具备授权。
请求使用只读 SELECT 子集，可包括显式列、别名、等值 INNER/LEFT JOIN、参数化常量比较、GROUP BY、COUNT 和 ORDER BY。输入文件中的 sql 是有业务用途的常规查询。LEFT JOIN 中不在授权范围的匹配产生无匹配的空字段，同一区有授权且确实匹配的行保留。授权采用 at 左闭右开且以后到的修订控制同一授权项。

## 本地资料

使用 Python 3.11.2 和内置 SQLite 3.40.1。以下命令从记录创建关系库，目标文件需要尚不存在。各角色可以作为独立 SQLite 文件读取。

~~~sh
python3 seed_db.py fixtures/shared-code-and-scope-revocation main-local.db --role main
~~~

使用 Python 的 sqlite3 打开新文件即可访问原始表。用 python3 -m unittest -v 核对资料可装载、表与原始记录一致、SQLite 可同时打开读写连接。后续事件保存在各自的原始记录中。
