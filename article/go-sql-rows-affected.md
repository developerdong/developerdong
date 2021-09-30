# Go中执行SQL UPDATE语句时RowsAffected()的返回值

今天在使用sql库执行UPDATE语句后，
获取RowsAffected()时犯了一个错误，在这里记录一下。

```sql
UPDATE students SET name=? WHERE id=?;
```

代码中的SQL语句类似于上面这个例子。
本意是想根据id在MySQL中找到记录进行更新，
然后通过`Result.RowsAffected()`来获取受影响的行数，
如果行数为0，则返回记录不存在的错误。
但测试时发现如果新name值和旧name值相同，
也就是说记录没有变化，
则`Result.RowsAffected()`返回值也为0。
所以如果想区分“找不到记录”、“记录未变化”这两种情况，
就需要主动地通过SELECT语句来确认记录是否存在。