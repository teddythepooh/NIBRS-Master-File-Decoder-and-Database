# NIBRS Master Files Decoder 

### Requirements
- Python 3.11.3; `requirements.yaml`
- (optional) Amazon Ubuntu EC2; Amazon S3 

### About
Through the FBI Crime Data Explorer (CDE), the FBI releases National Incident-Based Reporting System (NIBRS) data every year dating back to 1991. These so-called master files are the most comprehensive and centralized resource for *incident-level* public safety data, including but not limited to crime; arrest; and victim segments for over 13,000 law enforcement agencies across the United States. The data is in a fixed-length ASCII text format: every n characters is a distinct column, with the first two characters delineating between the different data segments. There exists one master file per year. The schemas for each segment can be found in the FBI CDE (see the yellow circle in the screenshot below): they are enumerated in a poorly-scanned document from 1995. For 2023, the data for which is to be released some time in fall 2024, the FBI has released the documentation as a proper .pdf entitled "2023.0 NIBRS Technical Specification:" https://le.fbi.gov/informational-tools/ucr/ucr-technical-specifications-user-manuals-and-data-tools. 

**My goals are to**
1. **develop a tool to seamlessly decode \& extract NIBRS data segments;**
2. **make the mechanism publicly available for others;**
3. **host the decoded segments in AWS S3;**
4. (in-process) **process and harmonize the data onto a PostgreSQL database, using either an EC2 or Amazon RDS instance;**
5. (in-process) **create a dashboard to visualize agency-level public safety statistics and reporting compliance;**
6. (in-process) **and explore ethical \& responsible machine learning applications in policing, like predicting clearances of crime in the spirit of resource allocation.**

![Screenshot 2024-09-05 011353](https://github.com/user-attachments/assets/6a2cb0be-3eb4-43df-893a-8c4768189c79)

In my pursuit of these goals, I hope to
1. build data engineering skills through an end-to-end ETL pipeline;
2. get comfortable with the AWS ecosystem, both interactively and from Python;
3. and expand my PostgreSQL knowledge, specifically by interacting with a database through SQLAlchemy (and psycopg2). I prefer to use psycopg2 only to the extent possible, but some pandas/polars methods expect a SQLAlchemy connection to interact with the database.

### Project Folder Structure
1. `src/` is the bread-and-butter of this repo: the scripts in this directory orchestrate the ETL pipeline.
2. `src/utils/` and `src/db_design/` contain the modules I developed—and continue to refine—to abstract away the core functionalities of `src/`. While `src/utils/` is specifically for the decoder tool and AWS integration features, `src/db_design/` is for all things SQL with and without Python.
2. `configuration/` is simply where I store configurable parameters as .yaml files.
3. `fbi_api_wrapper/` *will* be a Python wrapper for the FBI Crime Data API, from which I will extract agency information (e.g., agency name, population, etc.). This is technically available in the NIBRS master files from the so-called batch header segment, but the API exports are orders of magnitude more standardized and reliable. In that regard, the motivation of decoding the NIBRS master file is the fact that their aggregation is at the incident-level; in contrast, the FBI API's smallest aggregation is at the annual agency level.

### Instructions (for decoder tool only)
1. In the terminal, clone this repo and navigate to the parent directory (i.e., the same directory as this README).
1. Download the NIBRS ASCII text file from the FBI CDE, then store it in `raw_data/`; at this point, it should be a zipped file (e.g., `nibrs-2022.zip`) that is around 500 MB in size. You can unzip it, but the bash script below will handle the unzipping if needed. In that regard, it should get unzipped as `${data_year}_NIBRS_NATIONAL_MASTER_FILE_ENC.txt`. Such is the case for all years through 2022, and I presume the same applies to the 2023 master file when it is released in fall 2024.
2. To send the decoded data to an S3 bucket, as defined in `configuration/col_specs.yaml`, store your secrets as environment variables: `region_name`, `aws_access_key_id`, and `aws_secret_access_key`. Note that I set up my bash script to send the decoded data directly to my S3 bucket. However, if you would like to store them locally, delete line 39 in `main.sh` (i.e., the `to_aws_s3` flag) before proceeding.
3. Activate virtual environment.
4. Run `bash main.sh`.
![image](https://github.com/teddythepooh/NIBRS-Master-Files-Decoder/blob/aws_integration/images/nibrs_decoder_implementation.png)
5. The decoded data segments are now in Amazon S3.
![image](https://github.com/teddythepooh/NIBRS-Master-Files-Decoder/blob/aws_integration/images/s3_bucket.png)

### AWS Resources
This section is where I will store my AWS learning resources for future reference.
1. AWS S3 pricing: https://aws.amazon.com/s3/pricing/
2. AWS S3 intro: https://www.youtube.com/watch?v=tfU0JEZjcsg
3. IAM: https://stackoverflow.com/questions/46199680/difference-between-iam-role-and-iam-user-in-aws
4. How to SSH into EC2 from VS Code: https://stackoverflow.com/questions/56996544/vs-code-remote-ssh-to-aws-instance
