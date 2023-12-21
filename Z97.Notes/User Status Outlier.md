``` SQL
select * from query."WorkOrderHeader_Temporal"

 where "OrderNumber" =  '000101590631'

-- AND '2023-07-18 18:00:00.000' >= "EffectiveFromDate"

-- AND '2023-07-18 18:00:00.000' <= "EffectiveToDate"
;
```

``` SQL
SELECT * FROM WMKPI."ScheduledMaintenanceRatio"

WHERE "OrderNumber" = '000101590631'

```



![[Pasted image 20231220074304.png]]

Root Cause: Missing Status Change Documents in SAP resulting from S4 HANA when changes were made to work order status profiles at the DB Level.