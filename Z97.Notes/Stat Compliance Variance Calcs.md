tags:
	#DataManipulation
	 #StatutoryCompliance
links
	[[030.005 Statutory Compliance]]

# Window Functions To Calculate Variances

![[Pasted image 20231211105534.png]]

```SQL
    , IFF(T1."OrderNumber" IS NULL, '0', ROW_NUMBER() OVER (PARTITION BY
        T1."ReportingWeek"
        , T1."CompanyCode"
        , T1."Plant"
        , T1."Department"
    ORDER BY "AbsoluteDifferenceBetweenPlannedAndComplete" DESC)) AS "LargestVarianceByDepartment"

    , IFF(T1."OrderNumber" IS NULL, '0', ROW_NUMBER() OVER (PARTITION BY
        T1."ReportingWeek"
        , T1."CompanyCode"
        , T1."Plant"
        , T1."Department"
        , T1."MainWorkCentre"
    ORDER BY "AbsoluteDifferenceBetweenPlannedAndComplete" DESC)) AS "LargestVarianceByMainWorkCentre"
```