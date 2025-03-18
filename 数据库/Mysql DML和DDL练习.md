### JOIN

左连接

右连接

内连接（inner join): mysql 默认的join 用的就是内连接

全连接 (full join) 

自连接：将表与自身进行连接，用于在同一表中进行关联查询。

```mysql
SELECT column_name(s)
FROM table1 T1, table1 T2
WHERE condition;
```



### union 和union all的区别

都是对结果集的行进行合并，但是union all不会去重，union会去重。



### 聚合函数

max(), min(),count(),avg(),sum()  注意：count时要考虑场景要不要去重

group by:通常与上面的函数配合使用；

having: where 不能用于聚合函数中，所以用having 替换

```sql
// 查询某个字段存在两条以上的记录的所有行
select asset_no ,count(asset_no) assetNoNum from acflow_business_common.company_confirmed_asset_record ccar group by asset_no having assetNoNum>1

```



### 条件函数

#### if语句

if(condition,A,B) 如果condition成立，则A，否则B

```mysql
select username ,if(sex=1,'男','女') as sex from user 
例2
select if(age>=25,'25岁及以上','25岁以下') age_cut, count(device_id) number from user_profile group by age_cut
```

#### case语句

```mysql
select device_id,gender,
case
    when age<20 then '20岁以下'
    when age<25 then '20-24岁'
    when age>=25 then '25岁及以上'
    else '其他'
end age_cut
from user_profile;
```

case 和 count 或者 sum语句一起用

```sql
SELECT
        count(1) AS totalCount,
        SUM(f.plat_fee) AS totalAmount,
        COUNT(CASE WHEN tr.state not in ('thePayerRefusesConfirm','supplierRefusedToSignCashAgreement','fundersApprovalReject','loanFail','bankEntPayAcctAddingReject','theFinancingFailedBecauseTheAssetMatured') THEN 1 END) AS platFeeCount,
        SUM(CASE WHEN tr.state not in ('thePayerRefusesConfirm','supplierRefusedToSignCashAgreement','fundersApprovalReject','loanFail','bankEntPayAcctAddingReject','theFinancingFailedBecauseTheAssetMatured') THEN f.plat_fee ELSE 0 END) AS platFeeAmount
        <include refid="platFeeLedgerFrom"/>
        <include refid="platFeeLedgerWhere"/>
```

