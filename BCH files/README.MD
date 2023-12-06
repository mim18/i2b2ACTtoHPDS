<b>Below are the general steps to populate BCH data in HPDS in GIC format.</b></br>
1. Stage GIC ontology data, In addition populate source system specific mappings for lab result,Medication and visit type as provided by BCH.</br>
2. Extract fact data using GIC ontology in database table HPDS_DATA_LATEST.</br>
3. Extract java bin from the data-set created above using "Load HPDS Data From RDBMS Jenkins job".</br>
4. Upload it in the App.</br>

<b>Stage GIC ontology data, in addition populate source system specific mappings for lab result,Medication and visit type provided by BCH.</b></br>

[Download GIC ontology data files](https://github.com/hms-dbmi/i2b2ACTtoHPDS/tree/master/i2b2ACT%20to%20HPDS%20Data)<br> 
[Download system specific mappings](https://github.com/hms-dbmi/i2b2ACTtoHPDS/tree/BCH_ACT_Mappings/BCH%20files/DataFiles)<br>
[Script to create database Objects needed](https://github.com/hms-dbmi/i2b2ACTtoHPDS/blob/BCH_ACT_Mappings/BCH%20files/PRC_CRT_TABLES_ACT_HPDS_LOAD.sql)<br>
[Script to create ETL package needed](https://github.com/hms-dbmi/i2b2ACTtoHPDS/blob/BCH_ACT_Mappings/BCH%20files/TM_DATA_ACT_LOAD_PKG.sql)

<b> Listed GIC ontology data files are currently loaded.</b></br>

Database Table Name|File
-----|-------  
ACT_ICD10CM_DX_2018AA_HPDS|ACT_ICD10CM_DX_2018AA_HPDS.dsv
GIC_BIOSAMPLES_HPDS|GIC_BIOSAMPLES_HPDS.dsv
NCATS_LABS_HPDS|NCATS_LABS_HPDS.dsv
ACT_CPT_PX_2018AA_HPDS|ACT_CPT_PX_2018AA_HPDS.dsv
ACT_MED_ALPHA_HPDS|ACT_MED_ALPHA_HPDS.dsv
NCATS_DEMOGRAPHICS_HPDS|NCATS_DEMOGRAPHICS_HPDS.dsv
NCATS_VISIT_DETAILS_HPDS|NCATS_VISIT_DETAILS_HPDS.dsv

<b> Listed source system specific data mapping are loaded for BCH.</b></br>

Database Table Name|File
-----|------- 
a_lab_cd_act_bch_map|insert_A_LAB_CD_ACT_BCH_MAP.sql
a_ncats_visit_details_map|insert_A_NCATS_VISIT_DETAILS_MAP.sql
a_med_cd_act_bch_map|a_med_cd_act_bch_map.csv
 
<b> Scripts to extract fact data using GIC ontology into database table HPDS_DATA_LATEST.</b>

Database Procedure|Description
---------|----------
TM_DATA_ACT_LOAD_PKG.Run_MAP_Data_Load|populates the data mapping table ACT_BCH_ONTOLOGY_MAP, which creates mapping between your source system and GIC ontology mapping, depending upon the concept_cd naming convention used in your source system you might have to tweak the matching logic if no match is found for any of the ontology nodes.
TM_DATA_ACT_LOAD_PKG.Run_EXTRCT_HPDS_Data|extracts HPDS data in GIC format in HPDS_DATA_LATEST table.
</br> 
<b> Extract java bin from the data-set created above using Load HPDS Data From RDBMS </b>
Jenkins job needs listed required files for java bin extraction.
 
- sql.properties - with oracle database connect string
datasource.password=< your password >
datasource.user=< your user >
datasource.url=< your db connection string (currently only oracle) sampleformat jdbc:oracle:thin:@aaaabbbb.us-east.rds.amazonaws.com:1521/ORCL >
- loadQuery.sql - Modify to as listed.
SELECT PATIENT_NUM, CONCEPT_PATH, NVAL_NUM, TVAL_CHAR,START_DATE FROM HPDS_DATA_LATEST ORDER BY CONCEPT_PATH, PATIENT_NUM,START_DATE
</br> 
After the extraction process completes it creates listed files in /usr/local/docker-config/hpds

columnMeta.javabin</br> 
allObservationsStore.javabin</br> 
encryption_key</br> 
</br> 

<b>tar the above files </b>
</br>
tar -zcvf hpds_phenotype_gic_ALS-23456_10_02_2021_07_52_11_963514c.tar.gz  allObservationsStore.javabin columnMeta.javabin encryption_key
 
</br>
<b>Login to the Application server.</b>

Download the tar file in /usr/local/docker-config/hpds - file eg. hpds_phenotype_gic_ALS-23456_10_02_2021_07_52_11_963514c.tar.gz
- and Untar the file.

sudo tar xvzf hpds_phenotype_gic_ALS-23456_10_02_2021_07_52_11_963514c.tar.gz

<b>Re-Start the Start PIC-SURE Jenkins job</b>


## Frequntly Asked Questions </br>

<b>Question: Are there any guidelines to load data?</b><br>
Answer: [Upload data guide lines ](https://github.com/hms-dbmi/i2b2ACTtoHPDS/tree/master)

<b>Question: Are there any issues extracting large dataset?</b><br>
Answer: If the extraction job runs out of memory, increase heapsize parameter in the Jenkins jobs.

<b>Question: How should tval_char and nval_num be populated</b><br>
Answer: 

<b>QA test searches to vaildate all the nodes got loaded correctly in GIC</b><br>

GIC	Node|Filter type	|Search Criteria 
------------|---------------|----------------
ACT Procedures CPT	|No Values|search "Acetaminophen level" - retired CPT code - select ACT_Procedures_CPT tab
ACT Diagnosis ICD-10	|No values	|search  "J00 Acute nasopharyngitis [common cold]" - select ACT_Diagnosis_ICD_10 tab
ACT Medications	|No values	|search for "Histrelin" in ACT_Medications tab, select "H [Preparations], Restrict By Value Histrelin" - shows with categorical filter. Search for  "Aspirin" in ACT_Medications shows up as a numeric filter
ACT Lab Test Results	|Both numeric and categorical values	|search for "GGT" in ACT_Lab_Test_Results -select option"GGT: GGT in Serum or Plasma" -it shows with categorical filter. Search for  "Albumin" in ACT_Lab_Test_Results shows up as a numeric filter
NucleicAcid	|All are categorical values|search "NucleicAcid" select bio_specimens tab
Plasma	|Numeric filter 	|search "plasma" select bio_specimens tab
Blood	|Numeric filter 	|search "Blood" select bio_specimens tab
CSF	|Numeric filter 	|search "CSF" select bio_specimens tab
HumanTissue	|Numeric filter 	|search "Tissue" select bio_specimens tab
Gnome	|All are categorical values	|search "True"
Sex	|All are categorical values	|search "sex" and select act_demographics tab
Hispanic	|All are categorical values	|search "Hispanic" and select act_demographics tab
Race	|All are categorical values	|search "race" and select act_demographics tab.
Demographic Age	|Numeric filter 	|search "Age" and select act_demographics tab

