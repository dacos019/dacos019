select * 
from HSLABMEDECON.IDM_MemberSummary_Dev
sample 100 


-	What is the PMPM for the TN operational market for full year 2019. 
SELECT
EXTRACT(YEAR FROM CapDate) AS CapYear,
COUNT(MemberID) AS MemberCount,
SUM(ExpenseTotalClaims) TotalPaid,
TotalPaid/MemberCount AS PMPM
FROM HSLABMEDECON.IDM_MemberSummary_Dev
WHERE EXTRACT (YEAR FROM CapDate) IN ('2019')
AND OperationalMarket = 'TN'
GROUP BY EXTRACT(YEAR FROM CapDate)


-	What’s the inpatient PMPM for the AL and TX operational markets for quarter three 2020? 
SELECT
COUNT (MemberID) AS MemberCount,
SUM (LVL1_Inpatient_HP) AS InpatientVisits,
InpatientVisits/MemberCount AS PMPM
FROM HSLABMEDECON.IDM_MemberSummary_Dev
WHERE BeginDate  between '2020-01-01' and '2020-03-31'
AND OperationalMarket IN ('AL','TX') 

-	Match NPI # to Provider First, Last Name

Select hpp.ProviderFirstName,
hpp.ProviderLastName_LegalName,
cms.NPI 
from HSLABMEDECON.HPP_CompetitorNetwork_dev hpp
left join CMS_CORE_V.CMS_NPPES_NPIDATA cms
on hpp.NPI = cms.NPI 


-	Total Paid Calculation (In Network vs Out of Network)

Select OperationalMarket,
sum(TotalPaid),
Case when ParFlag = 'P' then
'In-Network'
when ParFlag = 'N' then
'Out of-Network'
when ParFlag = 'U' then
'Unknown'
End as Parflag
--'01' as Paid_Month
from OSS_PROVISIONING_V.sdoGBSAClaim 
where Extract(Year from PaidDateKey) = '2019'
and PlanType <> 'U'
and OperationalMarket <> 'U'
and ProductType = 'Medicare'
--and Extract(Month from PaidDateKey) = '1'
group by OperationalMarket,ParFlag
order by OperationalMarket

- Drop tables is best practice for temp tables to combine

DROP TABLE IP_TenetStage_1;
      CREATE MULTISET VOLATILE TABLE IP_TenetStage_1 AS
      (SELECT
    FacilityAddressLine1, 
    FacilityCity,
    FacilityCounty,
    NPI
    FROM HSLABMEDECON.LS_TENET_2019CTRCT_FACILITIES a
    LEFT JOIN IP_claims b
    ON a.NPI= b.ProviderNPI
    WHERE FacilityCity <> 'SAN ANTONIO' ) with data primary index (NPI) on commit preserve rows;

ALWAYS INCLUDE FOR DROP TABLES
 BEGINNING
CREATE MULTISET VOLATILE TABLE IP_TenetStage_1 AS

ENDING 
with data primary index (NPI) on commit preserve rows;


HSLABMEDECON.Clarify_Comp_Provider_data


=IFERROR(VLOOKUP(TEXT(A2,0),Sheet4!$B$2:$B$53,1,FALSE),VLOOKUP(VALUE(A2),Sheet4!$B$2:$B$53,1,FALSE))



--drop table as Tenet_Readmit_cnt;
--CREATE MULTISET VOLATILE TABLE Tenet_Readmit_cnt AS (
select distinct 
a.providernpi,
a.providername, 
a.readmissionprovidername, 
a.sourcedatakey, 
A.ProviderID,
a.claimid, 
a.ReadmissionProviderName,
a.ReadmissionProviderNPI,
a.TotalPaid totalpaidamt,
c.Updated_totalpaid,
a.ReadmissionAdmissionDate,
a.ReadmissionDOSYear,
a.OperationalMarket,
a.Hcode,
a.PBP,
count(distinct a.ClaimID) admits 
, case when a.readmissionprovidername is not null 
      and a.providername = a.readmissionprovidername
      then 1 else 0 end Readmit_count
from HSLABMEDECON.HOSPITAL_EVENTS_DETAIL A
Inner join hslabmedecon.TENETHOSPITALS_DA B
ON A.Providernpi=b.NPIs
left join (
            select distinct claimid ,  sum(totalpaid) Updated_totalpaid
            from OSS_PROVISIONING_V.sdoGBSAClaim 
            where Level1Bucket = 'Inpatient' 
            and ProductType = 'medicare'
            and dosbegin between '2020-01-01' and '2020-12-31'  
            and FinalPaidClaim = 'y' 
            --and  claimid = '20200801038480199' 
            group by 1 ) c 

--inner JOIN OSS_PROVISIONING_V.sdoGBSAClaim C
ON A.ClaimID = C.CLAIMID
where a.Level1Bucket = 'Inpatient' 
and a.ProductType = 'medicare'
and a.DOSYear = '2020' 
and a.drgcode not in ('880',      '881',     '882',     '883',      '884',     '885',     '886',     '887',     '894',     '895',     '896',      '897',     '945',     '946')
group by a.providernpi,
a.providername, 
a.readmissionprovidername, 
a.sourcedatakey, 
A.ProviderID,
a.claimid, 
a.ReadmissionProviderName,
a.ReadmissionProviderNPI,
a.TotalPaid,
c.Updated_totalpaid,
a.ReadmissionAdmissionDate,
a.ReadmissionDOSYear,
a.OperationalMarket, 
A.HCode,
A.PBP,
Readmit_count ; 



--MATCH NPI TO HOSPITAL CCN
SELECT * FROM HSLABMEDECON.AHD_CostReportSummary WHERE CMSCertificationNumber = '510013'

--SEARCH FOR HOSPITALS
SELECT * FROM REFDATA_CORE_V.AHD_COSTREPORTSUMMARY

--SEARCH ANY DATABASE

SELECT DBC.columnsv


--selecting multiple CPT Codes

select distinct A.memberid,
A.procedurecode
from OSS_PROVISIONING_V.sdogbsaclaim A
where finalpaidclaim = 'y' and producttype = 'medicare'
and DOSbegin between '2020-06-01' and '2021-05-31' 
AND (
(A.ProcedureCode BETWEEN 'A4361' AND 'A4435')
OR (A.ProcedureCode BETWEEN 'A5051' AND 'A5093')
OR (A.PROCEDURECODE BETWEEN 'A4405' AND 'A4435')
OR (A.PROCEDURECODE BETWEEN 'L5610' AND 'L5699')
OR (A.PROCEDURECODE BETWEEN 'L5710' AND 'L5966')
OR (A.PROCEDURECODE BETWEEN 'E0950' AND 'E1298')
OR (A.PROCEDURECODE BETWEEN 'K0001' AND 'K0108')
OR (A.PROCEDURECODE BETWEEN 'E0250' AND 'E0329')
OR (A.PROCEDURECODE BETWEEN 'E0621' AND 'E0642')
OR (A.PROCEDURECODE BETWEEN 'A4611' AND 'A4613')
OR (A.PROCEDURECODE BETWEEN 'E0465' AND 'E0467')
OR (A.PROCEDURECODE ='A4483')
OR (A.PROCEDURECODE ='A4422')
OR (A.PROCEDURECODE ='A4335')
OR (A.PROCEDURECODE ='E0277')
) 
group by a.memberid,a.procedurecode








--Covid Claims: Claim Level Detail 
select memberid , claimid , dosbegin 
, extract(month from dosbegin) DOSBEGIN_Month
, dosend
, paiddatekey , extract(month from Paiddatekey) Paiddatekey_Month
, trim(extract(year from Paiddatekey)) || '_' || trim(extract(month from Paiddatekey))  Year_Month_PaidDate
, DRGCode , DRGweights , operationalmarket , regulatorymarket , producttype 
, level1bucket , level2bucket , level3bucket , allowedamt, totalpaid, covid19_confirmed , covid19_testing
from hslabmedecon.covid19_claims
where PaidDateKey <= '2021-06-30'
and DOSbegin <= '2021-06-30'
and ProductType = 'medicare' ; 

--Confirmed Expense
select sum(allowedamt) Treatment_Allowed
from hslabmedecon.covid19_claims
where PaidDateKey <= '2021-06-30'
and DOSbegin <= '2021-06-30'
and ProductType = 'medicare' 
and Covid19_Confirmed = 'y' ;  

--Testing Expense 
select sum(allowedamt) Testing_Allowed
from hslabmedecon.covid19_claims
where PaidDateKey <= '2021-06-30'
and DOSbegin <= '2021-06-30'
and ProductType = 'medicare' 
and Covid19_Confirmed = 'n'
and Covid19_Testing = 'y' ; 

--Total Covid Related Expense 
select sum(allowedamt) TotalCovid_Allowed
from hslabmedecon.covid19_claims
where PaidDateKey <= '2021-06-30'
and DOSbegin <= '2021-06-30'
and ProductType = 'medicare'  ; 


--Check of Payment amounts and admits for hospitals
select ProviderNPI , providerID 
, count(distinct claimID) Claims
, sum(totalpaid) TotalPaid
from OSS_PROVISIONING_V.sdogbsaclaim a
where a.Level1Bucket = 'Inpatient' 
and a.ProductType = 'medicare'
and finalpaidclaim = 'y' 
and dosbegin between '2020-01-01' and '2020-12-31'  
and a.drgcode not in ('880',      '881',     '882',     '883',      '884',     '885',     '886',     '887',     '894',     '895',     '896',      '897',     '945',     '946') 
and providerNPI in (
                      select distinct NPIs
                      from HSLABMEDeCON.DA_TENETHOSPITALLIST) 
group by 1,2



drop table Ortho_Claims_Tenet;
CREATE MULTISET VOLATILE TABLE Ortho_Claims_Tenet AS

(select distinct
A.claimid, 
A.memberid,
A.PROVIDERNPI,
a.providerid,
a.totalpaid,
a.operationalmarket,
a.drgcode,
a.drgdesc,
b. Facility
where FinalPaidClaim = 'y'
AND ProductType = 'MEDICARE'
and drgcode between '438' and '517'
and dosbegin BETWEEN '2020-01-01' and '2020-12-31'
from OSS_PROVISIONING_V.sdoGBSAClaim A
left join ( 
select
facility, facility city,ProviderID
from HSLABMEDeCON.DA_TENETHOSPITALLIST
group by 1,2,3) b
ON A.ProviderID=B.ProviderId
group by A.claimid, 
A.memberid,
A.PROVIDERNPI,
a.providerid,
a.totalpaid,
a.operationalmarket,
a.drgcode,
a.drgdesc,
b. Facility)
with data primary index (ProviderID) on commit preserve rows;



DROP TABLE ADE_CLAIMS;
CREATE  MULTISET VOLATILE TABLE ADE_CLAIMS AS
(
select  A.clm_ID, A.SYS_ICD_DIAGS_CD
FROM reporting_v.cdo_clm_diag_cd A
WHERE A.SYS_ICD_DIAGS_CD BETWEEN 'Y63.6' AND 'Y63.9'
OR (A.SYS_ICD_DIAGS_CD BETWEEN 'Z91.12' AND 'Z91.14')
or (A.SYS_ICD_DIAGS_CD BETWEEN 'T36.0' AND 'T65.0')
) with data primary index (clm_ID) on commit preserve rows;

select * from ADE_CLAIMS;

DROP TABLE ADE_MEMBERS;
CREATE  MULTISET VOLATILE TABLE ADE_MEMBERS AS
( select distinct memberid,SYS_ICD_DIAGS_CD,claimid,OPERATIONALMARKET,level2bucket
from ADE_CLAIMS a 
left join REPORTING_V.sdo_gbsa_clm b 
on a.clm_id = b.claimID 
where b.FinalPaidClaim = 'Y' 
and operationalmarket ='nc'
and level2bucket = 'Facility Outpatient ER'
and DOSbegin between '2019-01-01' and '2021-06-30'
and b.ProductType = 'medicare'
group by 1,2,3,4,5
) with data primary index (memberid) on commit preserve rows;


--Total Payments
select distinct 
a.providernpi,
a.providername, 
a.readmissionprovidername, 
a.sourcedatakey, 
A.ProviderID,
a.claimid, 
a.ReadmissionProviderName,
a.ReadmissionProviderNPI,
a.TotalPaid totalpaidamt,
c.Updated_totalpaid,
a.ReadmissionAdmissionDate,
a.ReadmissionDOSYear,
a.OperationalMarket,
a.Hcode,
a.PBP,
count(distinct a.ClaimID) admits 
, case when a.readmissionprovidername is not null 
      and a.providername = a.readmissionprovidername
      then 1 else 0 end Readmit_count
from HSLABMEDECON.HOSPITAL_EVENTS_DETAIL A
Inner join hslabmedecon.TENETHOSPITALS_DA B
ON A.Providernpi=b.NPIs
left join (
            select distinct claimid ,  sum(totalpaid) Updated_totalpaid
            from OSS_PROVISIONING_V.sdoGBSAClaim 
            where ProductType = 'medicare'
            and dosbegin between '2020-01-01' and '2020-12-31'  
            and FinalPaidClaim = 'y' 
            --and  claimid = '20200801038480199' 
            group by 1 ) c 

--inner JOIN OSS_PROVISIONING_V.sdoGBSAClaim C
ON A.ClaimID = C.CLAIMID
where a.ProductType = 'medicare' 
and a.DOSYear = '2020' 
group by a.providernpi,
a.providername, 
a.readmissionprovidername, 
a.sourcedatakey, 
A.ProviderID,
a.claimid, 
a.ReadmissionProviderName,
a.ReadmissionProviderNPI,
a.TotalPaid,
c.Updated_totalpaid,
a.ReadmissionAdmissionDate,
a.ReadmissionDOSYear,
a.OperationalMarket, 
A.HCode,
A.PBP,
Readmit_count ; 


--Checks to See Duplicate Records
select  provider_npi,ParentOrganization, count(hcode),hcode
from HSLABMEDECON.Clarify_CompetitorNetwork
having count(hcode) >1
group by 1,2,4
order by provider_NPI


--UPLOAD OF OCT
select *
from HSLABMEDECON.Clarify_Extract_Update_DA_Oct


---Only Providers not under Cigna
SELECT DISTINCT sc.pROVIDER_NPI,TIN,HCODE,ProviderName,ProviderCredentialText,Provider_Address,PARENTORGANIZATION,plantype,medicarespecialtydesc,TaxonomyDesc,Prov_Spec_Updated,pri_spec,CBSA_REGION,CBSA_CITY,CBSA_State
FROM hslabmedecon.Clarify_competitornetwork sc
WHERE NOT EXISTS(
    SELECT * FROM hslabmedecon.Clarify_competitornetwork sc2 
    WHERE sc2.PROVIDER_NPI = sc.PROVIDER_NPI AND sc2.Parentorganization = 'CIGNA' )
    and CBSA_City IN ('Knoxville' ,'Athens'  ,'Cleveland','Greeneville', 'Johnson City','Chattanooga','kingsport','morristown','newport','sevierville')
    and CBSA_State = 'TN'
-- AND provider_NPI ='1003149568'
   order by 1



--Pulls all Providers and Unions 4 states
DROP TABLE VA_MS_TN_AR_Clarify_Network;
CREATE  MULTISET VOLATILE TABLE VA_MS_TN_AR_Clarify_Network AS
(SELECT DISTINCT sc.pROVIDER_NPI,
TIN,
ProviderName,
ProviderCredentialText,
Provider_Address,
PARENTORGANIZATION,
plantype,
HCODE,
medicarespecialtydesc,
TaxonomyDesc,
Prov_Spec_Updated
,pri_spec,
CBSA_REGION,
CBSA_CITY,
CBSA_State,
CBSA_County,
provider_zipcode
FROM hslabmedecon.Clarify_competitornetwork sc
    WHERE CBSA_State IN ('VA','MS')
    And sc.CBSA_County IN ('Wise','Scott','Russell','Washington','DeSoto','Tunica','Marshall')

   
union
  

SELECT DISTINCT dc.pROVIDER_NPI,
TIN,
ProviderName,
ProviderCredentialText,
Provider_Address,
PARENTORGANIZATION,
plantype,
HCODE,
medicarespecialtydesc,
TaxonomyDesc,
Prov_Spec_Updated
,pri_spec,
CBSA_REGION,
CBSA_CITY,
CBSA_State,
CBSA_County,
provider_zipcode
FROM hslabmedecon.Clarify_competitornetwork dc
    WHERE CBSA_State IN ('AR','TN')
    )  with data primary index (provider_NPI) on commit preserve rows;


--Ordering hospitals with over 100 beds
select ProviderName, count(distinct hcode) total_hcode,medicare_certified_beds
from HSLABMEDECON.clarify_competitornetwork
group by ProviderName, medicare_certified_beds
order by Medicare_Certified_Beds
having Medicare_Certified_Beds > 100
