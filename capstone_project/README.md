# US I94 Immigration Insights 
### Data Engineering Capstone Project
<p align="center"><img src="udacity (1).png" style="height: 100%; width: 100%; max-width: 200px" /></p>

#### Project Summary


This Project creates a Data Lake type of ETL pipeline to process, clean and store data related to US I94 Immigration data. Data can be used to analyse immigration flow to and from US through different airports. It's used a star schema with a facts table an dimensional tables.

The project follows the follow steps:
* Step 1: Scope the Project and Gather Data
* Step 2: Explore and Assess the Data
* Step 3: Define the Data Model
* Step 4: Run ETL to Model the Data
* Step 5: Complete Project Write Up


#### Scope 



This projects aims to enrich the US I94 immigration data with further data such as demographics and temperature data to have a wider basis for analysis on the immigration data.
I developed a data pipeline that creates an analytics database for querying information about immigration into the U.S. 


Created an ETL pipeline for processing, cleaning and storing data related to US I94 immigration data, and country codes.  

Output of the ETL pipeline: processed data stored in Star schema model to parquet files. 
Tools: python, pandas, pyspark

#### Describe and Gather Data 


Project's data contains the following pieces:
* **data/18-83510-I94-Data-2016/**: US I94 immigration data from 2016 (Jan-Dec).
    * Source: https://travel.trade.gov/research/reports/i94/historical/2016.html
    * Description: I94_SAS_Labels_Descriptions.txt file contains descriptions for the I94 data.
        * I94 dataset has SAS7BDAT file per each month of the year (e.g. i94_jan16_sub.sas7bdat).
        * Each file contains about 3M rows
        * Data has 28 columns containing information about event date, arriving person, airport, airline, etc.
    * I94 immigration data example:
    * ![I94-immigration-data example](capstone_project/Udacity-DEND-Project-Capstone-I94ImmigrationData-20190812-2.png)
    * NOTE: This data is behind a pay wall and need to be purchased to get access. Data is available for Udacity DEND course.
    
* **data/i94_airport_codes.xlsx**: Airport codes and related cities defined in I94 data description file.
    * Source: https://travel.trade.gov/research/reports/i94/historical/2016.html
    * Description: I94 Airport codes data contains information about different airports around the world.
        * Columns: i94port, i94_airport_name
        * Data has 660 rows and 2 columns.
    * NOTE: I94 data uses its own codes for airports instead of using standard codes (like IATA). Therefore, I94 airport codes have been taken from I94 data description file and processed for ETL use.  
    * Airport Code example:
    * ![I94-AirportCode-data example](capstone_project/Udacity-DEND-Project-Capstone-I94AirportCodeData-20190813-4.png)

* **data/i94_country_codes.xlsx**: Country codes defined in US I94 Immigration data description file. 
    * Source: https://travel.trade.gov/research/reports/i94/historical/2016.html
    * Description: I94 Country codes data contains information about countries people come to US from.
        * Columns: i94cit, i94_country_code
        * Data has 289 rows and 2 columns.
    * NOTE: I94 data uses its own codes for countries instead of using ISO-3166 standard codes. Therefore, I94 country codes have been taken from I94 data description file and processed for ETL use.
    * Country Code example:
    * ![CountryCode-data example](capstone_project/Udacity-DEND-Project-Capstone-I94CountryCodeData-20190813-5.png)    
  
* **data/airport-codes.csv**: Airport codes and related cities.
    * Source: https://datahub.io/core/airport-codes#data
    * Description: Airpot codes data contains information about different airports around the world.
        * Columns: Airport code, name, type, location, etc.
        * Data has 48304 rows and 12 columns.
    * Airport Code example:
    * ![AirportCode-data example](capstone_project/Udacity-DEND-Project-Capstone-AirportCodeData-20190812-3.png)

* **data/iso-3166-country-codes.json**: World country codes (ISO-3166)
    * Source: https://github.com/lukes/ISO-3166-Countries-with-Regional-Codes
    ISO-3166-1 and ISO-3166-2 Country and Dependent Territories Lists with UN Regional Codes
    * ISO-3166: https://www.iso.org/iso-3166-country-codes.html
    * Country Code example:
    * ![CountryCode-data example](capstone_project/Udacity-DEND-Project-Capstone-CountryCodesData-20190804-4.png)
    
    
    
    
    
    
    ---------
###  Define the Data Model
#### Conceptual Data Model


I94 Immigration Insights data models is a star models consisting of 4 Dimensions table and 1 Fact table:
  * Dimensions tables:
      * admissions table
      * countries table
      * airports table
      * time table
  * Fact table:
      * immigrations table
      
ERD for the project:
<p align="center"><img src="capstone_project/Udacity-DEND-Project-Capstone-ERD-20190820v11_RAVI.png" /></p>



####  Mapping Out Data Pipelines

* First, ETL script reads in configuration settings (dl.cfg). Script also re-orders I94 inout files to process them in right order (Jan => Dec).
* ETL script takes input data (I94 data, I94 country data, I94 airport data, ISO-3166 country data, IATA airport data).
* Raw input data is read into pandas dataframe, and from there to Spark dataframe and stored into parquet staging files.
* Staging parquet files are read back to Spark dataframes and cleaned (when necessary) and some further data is extracted from the original data.
* Each star schema table is processed in order: admissions => countries => airports => time => immigrations
* Finally, data quality checks are run for each table to validate the output (key columns don't have nulls, each table has content). A summary of the quality check is provided and written in console.



## Data Dictionary for the project is described in **data_dictionary.json** file stored in the project root directory.





**Rationale for the tools selection:**
* Python, Pandas and Spark were natural choises to process project's input data since it contains all necessary (and easy to use) libraries to read, clean, process, and form DB tables.
* Since the data set was still limited, local and server storage was used in storing, reading, writing the input and output data. 
* Input data could have been stored in AWS without big problems (excluded in this project). 
* Output data could have been easily written to AWS after processing (excluded in this project). Experiences have shown that it's better to write parquet files locally first and only after that write them to cloud storage (as a bulk oparation) to avoid delays and extra costs caused by AWS S3.

**How often ETL script should be run:**
* ETL script should be run monthly basis (assuming that new I94 data is available once per month).

**Other scenarions (what to consider in them):**
* Data is 100x: 
    * Input data should be stoted in cloud storage e.g. AWS S3
    * Clustered Spark should be used to enable parallel processing of the data.
    * Clustered Cloud DB e.g. AWS Redshift should be used to store the data during the processing (staging and final tables).  
    * Output data (parquet files) should be stored to Cloud storage e.g. AWS S3 for easy access or to a Cloud DB for further analysis. AWS Redshift is very expensive for storing the data, so maybe some SQL DB (e.g. AWS RDS) should be used. 
    
* Data is used in dashboard and updated every day 07:00AM:
    * ETl script should be refactored to process only the changed inout information instead of processing all the inout files as it does now to minimise the used time and comouting resources.
    * Output data should be stored and updated in a Cloud DB (e.g. AWS RDS) to make it available all times for the dashboard.
    * Possibly this "always available" DB (serving the dashboard) would contain a latest sub-set of all available data to make it fast perfoming and easier to manage.

* DB is accessed by 100+ people:
    * The more people accessing the database the more CPU resources you need to get a fast experience. By using a distributed database you can improve your replications and partitioning to get faster query results for each user.
    * Potentially, some new tables could be created to serve the most used queries better.

**Potential further work:** 
* ETL pipeline script could be re-factored
    * make it more modular (split functions to separate files/classes)
    * combine functions to have fewer, more general purpose functions instead of several specific function per ETL steps 
    
* IATA airport data could be (semi-manually) mapped to I94 airport data to add more value for the analysis and enable further data merges.

* Other data e.g. daily weather data could be combined as inout data to provide insights about the weather immigrants experienced when they entered US. 
