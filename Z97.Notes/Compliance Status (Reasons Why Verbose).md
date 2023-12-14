tags:
	#SQL
	#CodeSnippets

[[Reasons for Non Compliance Stat Completion]]


```SQL

 , CASE

        WHEN "OpenStatutoryOrder" = 1 --Check If Reporting Week Has Order

            THEN

                CASE

                    WHEN T1."PlannedDate" <= T1."ReportingWeekStart"

                        THEN

                            CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 'Not Compliant Work Order Is Cancelled'

  

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusForwardSnapshot", '.*CRTD.*') = TRUE

                                    OR REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*CRTD.*') = TRUE

                                    THEN 'Not Compliant Work Order Still Created In Forward Or Backward Snapshot'

  

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*CRTD.*|.*REL.*') = TRUE

                                    AND "AbsoluteDifferenceBetweenPlannedAndComplete"

                                    OR "DifferenceBetweenPlannedAndReportingWeekStart" > 14  --<===== THIS IS THE SCENARIO ORDER 002200587772 Planned Date was OK Then Pushed Forward

                                    THEN 'Not Compliant Orders Basic Finish is Greater Then 14 Days From Planned Date'

  

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*TECO.*|.*CLSD.*') = FALSE

                                    AND DATEDIFF('Days', T1."PlannedDate", T1."ReportingWeekStart") > 14

                                    THEN 'Not Compliant Order Still Open & 14 Days Out Of Range of Planned Date'

  

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*TECO.*|.*CLSD.*') = TRUE

                                    AND DATEDIFF('Days', T1."PlannedDate", T1."ReportingWeekStart") > 14

                                    THEN 'Not Compliant Order Closed & 14 Days Out Of Range of Planned Date'

  

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*TECO.*|.*CLSD.*') = TRUE

                                    AND DATEDIFF('Days', T1."PlannedDate", T1."ReportingWeekStart") <= 14

                                    THEN 'Compliant Order Closed Within 14 Days of Planned Date'

  

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*REL.*') = TRUE

                                    AND DATEDIFF('Days', T1."PlannedDate", T1."ReportingWeekStart") <= 14

                                    THEN 'Compliant Order Open Within 14 Days of Planned Date'

  

                                ELSE 'Other 1234'

                            END

                    WHEN T1."PlannedDate" BETWEEN T1."ReportingWeekStart" AND T1."ReportingWeekFinish"

                        THEN CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 'Not Compliant Work Order Is Cancelled'

                                WHEN REGEXP_LIKE("WorkOrderSystemStatusBackwardSnapshot", '.*CRTD.*') = TRUE

                                    AND "AbsoluteDifferenceBetweenPlannedAndComplete" > 14  --<===== THIS IS THE SCENARIO ORDER 002200587772 Planned Date was OK Then Pushed Forward

                                    THEN 'Not Compliant Orders Basic Finish is Greater Then 14 Days From Planned Date'

                                ELSE 'Other4321'

                            END

                    WHEN T1."PlannedDate" > T1."ReportingWeekFinish"

                        THEN

                            CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 'Not Compliant Work Order Is Cancelled'

                                WHEN "DifferenceBetweenPlannedAndBasidFinishForward" NOT BETWEEN -14 AND 14

                                    THEN 'Not Compliant Orders Basic Finish is Greater Then 14 Days From Planned Date'

                                WHEN "DifferenceBetweenPlannedAndBasidFinishForward" BETWEEN -14 AND 14

                                    THEN 'Compliant Order Open Within 14 Days of Planned Date'

                                ELSE 'Not Compliant Other'

                            END

                    WHEN T1."MaintenancePlanNumber" = ''

                        THEN 'Not Compliant No Maintenance Plan'

                    ELSE 'Not Compliant Other'

                END

        WHEN "OpenStatutoryOrder" = 0

            THEN 'Compliant No Open Stat Work Orders For Reporting Week'

        ELSE 'Not Compliant Other'

    END AS "ComplianceStatus"

```