# mysql命令

### 查看链接：show processlist

![image-20210806072111978](.\images\image-20210806072111978.png)

- Command 列显示为“Sleep”的这一行，就表示现在系统里面有一个空闲连接。

**查看参数：show variables like 'transaction_isolation'**

```sql
mysql> show variables like 'transaction_isolation';
+-----------------------+----------------+
| Variable_name | Value |
+-----------------------+----------------+
| transaction_isolation | READ-COMMITTED |
+-----------------------+----------------+
```

