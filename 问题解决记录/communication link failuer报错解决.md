### 1. 报错场景

![](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20241015135648513.png)

确认账款操作执行后，等待10s会抛出异常，查看日志如上

确认账款的代码简化后主要业务逻辑如下

报错代码位置为“先确权场景”工作流节点报错 updateAssetExtField方法执行时连接失败

```java
public updateOpenTime(){
    // 将凭证开立时间回写到原始资产信息里
    Map<String, Object> extFieldMap = new HashMap<>();
    extFieldMap.put("voucherOpenTime", DateUtil.getDate("yyyy-MM-dd HH:mm:ss"));
    assetInfoProvider.updateAssetExtField(assetInfo.getId(), extFieldMap);
}
```

### 2.问题定位与解决

从报错信息来看为jdbc连接失败，并且提到了上一次从数据库接收到响应为10s之前。

1. 首先想到的是连接超时了，查看连接超时时间设置

   ```sql
   show variables like '%timeout%'
   ```

![image-20241016155833284](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20241016155833284.png)

​		看到连接超时时间刚好为10s，怀疑就是连接超时了，调大一点试试 

```sql
set global connect_timeout 60；
```

结果是没有效果，提示还是和之前一样，说明问题不在这；

2. sql执行connect_timeout设置的是连接超时时间，socket_timeout才是sql执行的响应时间，所以试了下更改socket_timetout更大一点，不过也是没有任何响应

   ```
   x jdbc:mysql://xxx:xxx/db?socketTimeout=60000
   ```

3. 查看官方文档发现了连接池的探活设置，连接池管理的所有连接，如果存在连接长时间处于空闲状态时，超过参数wait_timeout的值，mysql会关闭该连接，而连接池仍然认为该连接还是有效的，应用申请该连接时，就会报错。

   wait_timeout数据库中的值为28800s(8小时)，不太可能会超过，不过还是尝试改大一点，以及开启连接有效性设置

   ```
   testOnBorrow: true
   wait_timeout: 86400
   ```

4. 尝试了一系列的参数设置以后，都没有解决，应该不是数据库参数设置的问题，询问了DBA最近也没有更改过参数设置。回过头来仔细看代码和报错日志。发现同一段代码参数一样只有自动确权流程会报错，手动确权流程不会。那有可能就是节点之前的操作把表锁住了，定位以后果然是这个问题。

   ```java
   	@Transactional(rollbackFor = Exception.class)
       public void createRvsTsAsset(Long assetId, boolean confirmFlag) {
           // 资产信息
           AssetInfoVo assetInfo = assetProvider.queryById(assetId);
           // 账款校验
           confirmAssetValidate(assetInfo);
           // 判断确权流程是否先确权，如果是则发起先确权流程
           if (confirmFlag) {
               Long businessId = (long) (System.currentTimeMillis() + (Math.random() * 1000000));
               JSONObject variables = this.createAssetConfirmVariables(assetInfo, businessId);
               Map<String, Object> extFieldMap = new HashMap<>();
               if (businessId != null) {
                   extFieldMap.put("businessId", businessId);
               }
               // 这里会更新asset_ext表数据
               assetInfoProvider.updateAssetExtField(assetInfo.getId(), extFieldMap);
               try {
                   // 发起工作流流程
                   ProcessStartVo processStartVo = workflowNewProvider.startProcess(WfConstants.RVS_CASH_ASSET_CONFIRM_BEFORE_FLOW, variables);
               }catch (Exception e) {
                   log.error("启动工作流失败：", e);
                   assetInfoProvider.updateAssetInfoStatus(assetInfo.getId(), AssetStatusEnum.UNCONFIRMED, AssetCateEnum.MLF);
                   throw new BaseException("启动工作流失败,请稍后重试，或者刷新数据");
               }
           } else {// 后确权流程
               createTsAsset(assetInfo, AssetStatusEnum.UNFUNDED, TsAssetStatus.UNFUNDED);
           }
           //保存确认账款日期
           Map<String, Object> extFieldMap = new HashMap<>();
           extFieldMap.put("confirmAssetDate", LocalDate.now().toString());
           assetInfoProvider.updateAssetExtField(assetInfo.getId(), extFieldMap);
       }
   ```

   在startProcess启动工作流之前，有一个updateAssetExtField的操作，会将asset_ext表锁住，然后接着执行startProcess方法，也就是工作流启动方法，工作流启动后会自动往下执行节点的代码，需要一直等待工作流执行完成所有节点后才返回，而自动确权流程中的一个节点操作就是上面报错的代码，也是更新asset_ext表，此时表被锁住，只能等待行锁释放，startProcess方法就一直阻塞不返回，createRvsTsAsset方法也就不会往下执行，事务永远不会提交，造成死锁。

### 3. 总结

communication link failure出现这个问题造成的原因有很多，如果 `n milliseconds ago` 中的 `n` 如果是 0 或很小的值，则通常是执行的 SQL 导致异常退出引起的报错，查看代码解决。如果 n 是一个非常大的值（28800），很可能是因为这个连接空闲太久然后被中间 proxy 关闭了。调整以下参数基本能解决

```
connectTimeout=600000 // 表示的是数据库驱动(mysql-connector-java)与mysql[服务器]建立TCP连接的超时时间。
socketTimeout=600000 // 是通过TCP连接发送数据(在这里就是要执行的sql)后，等待响应的超时时间。
testOnBorrow: true // 每次执行sql前是否检测连接的有效性
spring.datasource.druid.maxWait=60000// 程序向连接池中请求连接时,超过maxWait的值后,认为本次请求失败
spring.datasource.druid.minIdle=5 // 最小连接数
spring.datasource.druid.minEvictableIdleTimeMillis=600000 // 最小空闲时间，如果连接池中空闲的连接数大于minIdle(最小空闲连接数)，并且那部分连接的空闲时间大于minEvictableIdleTimeMillis，连接将被关闭
spring.datasource.druid.maxEvictableIdleTimeMillis=900000 // 最大空闲时间，空闲时间超过这个的连接将被关闭
```