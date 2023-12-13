tags:
	#KeyPerformanceIndicatorProject 
	 #KeyPerformanceReasons
	  #GapReporting

links:
	[[030.005 Statutory Compliance]]


For Gap Reporting we need top show to explain to the users why a call is not compliant, this is especially different as the way we report on work orders has changed. 

The Denominator is any open work order that has a PMAct Type for Statutory Works, this is different as we are no longer looking at if the work order is scheduled in the reporting week. This way consumers can see the whole picture not just what is Scheduled within the reporting week.
### Reasons for Non Compliance
	1. Work Order Cancelled
	2. Work Order Not Released Prior to Reporting Week Start
	3. Work Order Is Not Executed
	5. The Variance for the Planned & Completion Date is greater than 14 days


# FIX

- [ ] Not Compliant Work Order Planned Date Is More Then 14 Behind Reporting Week Finish Date|

    
   ```SQL
    , CASE

        WHEN "OpenStatutoryOrder" = 1 --Check If Reporting Week Has Order

            THEN

                CASE

                    WHEN T1."PlannedDate" <= T1."ReportingWeekFinish"

                        THEN

                            CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 0 -- Not Compliant Work Order Is Cancelled

                                WHEN REGEXP_LIKE(T1."WorkOrderSystemStatusForwardSnapshot", '.*CRTD.*') = TRUE

                                    AND CURRENT_TIMESTAMP >= T1."ForwardSnapshot"

                                    THEN 0 -- Not Compliant Work Order is Not Released At Forward Snapshot

                                WHEN REGEXP_LIKE(T1."WorkOrderSystemStatusForwardSnapshot", '.*CRTD.*') = TRUE

                                    AND CURRENT_TIMESTAMP < T1."ForwardSnapshot"

                                    AND "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 1

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*CRTD.*') = TRUE

                                    THEN 0 -- Not Compliant Work Order is Not Released At Backwards Snapshot

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*NCMP.*') = TRUE

                                    THEN 0 -- Not Compliant Work Order is Not Executed

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*REL.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 1 -- Compliant Order Is Open & Within 14 Days Of Planned Date

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*REL.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" NOT BETWEEN -14 AND 14

                                    THEN 0 -- Not Compliant Order Is Open & Not Within 14 Days Of Planned Date

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*TECO.*|.*CLSD.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" NOT BETWEEN -14 AND 14

                                    THEN 0 -- Not Compliant Order Is Closed & Not Within 14 Days Of Planned Date

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*TECO.*|.*CLSD.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 1 -- Compliant Order Is Closed & Within 14 Days Of Planned Date

                                ELSE 0 -- Not Compliant Other

                            END

                    WHEN T1."PlannedDate" > T1."ReportingWeekFinish"

                        THEN

                            CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 0 -- Work Order Is Cancelled

                                WHEN "DifferenceBetweenPlannedAndComplete" NOT BETWEEN -14 AND 14

                                    THEN 0 -- Not Compliant Order Is Open & Not Within 14 Days Of Planned Date

                                WHEN "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 1 -- Compliant Order Is Open & Within 14 Days Of Planned Date

                                ELSE 0 -- Not Compliant Other

                            END

                    WHEN T1."MaintenancePlanNumber" = ''

                        THEN 0 --Not Compliant No Maintenance Plan

                    ELSE 0 -- Not Compliant Other

                END

        WHEN "OpenStatutoryOrder" = 0

            THEN 0 --Compliant No Open Stat Work Orders For Reporting Week

        ELSE 0 --Not Compliant Other

    END AS "CompliantWorkOrder"

    , CASE

        WHEN "OpenStatutoryOrder" = 1 --Check If Reporting Week Has Order

            THEN

                CASE

                    WHEN T1."PlannedDate" <= T1."ReportingWeekFinish"

                        THEN

                            CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 'Not Compliant Work Order Is Cancelled'

                                WHEN REGEXP_LIKE(T1."WorkOrderSystemStatusForwardSnapshot", '.*CRTD.*') = TRUE

                                    AND CURRENT_TIMESTAMP >= T1."ForwardSnapshot"

                                    THEN 'Not Compliant Work Order is Not Released At Forward Snapshot'

                                WHEN REGEXP_LIKE(T1."WorkOrderSystemStatusForwardSnapshot", '.*CRTD.*') = TRUE

                                    AND CURRENT_TIMESTAMP < T1."ForwardSnapshot"

                                    AND "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 'Compliant Order Is Open & Within 14 Days Of Planned Date'

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*CRTD.*') = TRUE

                                    THEN 'Not Compliant Work Order is Not Released At Backwards Snapshot'

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*NCMP.*') = TRUE

                                    THEN 'Not Compliant Work Order is Not Executed'

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*REL.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 'Compliant Order Is Open & Within 14 Days Of Planned Date'

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*REL.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" NOT BETWEEN -14 AND 14

                                    THEN 'Not Compliant Order Is Open & Not Within 14 Days Of Planned Date'

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*TECO.*|.*CLSD.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" NOT BETWEEN -14 AND 14

                                    THEN 'Not Compliant Order Is Closed & Not Within 14 Days Of Planned Date'

                                WHEN REGEXP_LIKE(T2."SystemStatus", '.*TECO.*|.*CLSD.*') = TRUE

                                    AND "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 'Compliant Order Is Closed & Within 14 Days Of Planned Date'

                                ELSE 'Not Compliant Other'

                            END

                    WHEN T1."PlannedDate" > T1."ReportingWeekFinish"

                        THEN

                            CASE

                                WHEN REGEXP_LIKE(T2."UserStatus", '.*CANC.*') = TRUE

                                    OR REGEXP_LIKE(T1."WorkOrderUserStatusForwardSnapshot", '.*CANC.*') = TRUE

                                    THEN 'Not Compliant Work Order Is Cancelled'

                                WHEN "DifferenceBetweenPlannedAndComplete" NOT BETWEEN -14 AND 14

                                    THEN 'Not Compliant Order Is Open & Not Within 14 Days Of Planned Date'

                                WHEN "DifferenceBetweenPlannedAndComplete" BETWEEN -14 AND 14

                                    THEN 'Compliant Order Is Open & Within 14 Days Of Planned Date'

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