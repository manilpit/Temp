## Documentation at azure:
https://learn.microsoft.com/en-us/azure/mysql/flexible-server/how-to-create-manage-databases

# Prerequisites:
Before completing the tasks, you must have

Created an Azure Database for MySQL Flexible Server instance using Azure portal (https://learn.microsoft.com/en-us/azure/mysql/flexible-server/quickstart-create-server-portal)
or Azure CLI. (https://learn.microsoft.com/en-us/azure/mysql/flexible-server/quickstart-create-server-cli)
Sign in to Azure portal.(https://portal.azure.com/)

# List your databases
To list all your databases on Azure Database for MySQL Flexible Server:

Open the Overview page of your Azure Database for MySQL Flexible Server instance.
Select Databases from the settings on left navigation menu.

# Create a database
To create a database on Azure Database for MySQL Flexible Server:

Select Databases from the settings on left navigation menu.
Click on Add to create a database. Provide the database name, charset and collation settings for this database.
Click on Save to complete the task.


# Delete a database
To delete a database on Azure Database for MySQL Flexible Server:

Select Databases from the settings on left navigation menu.
Click on testdatabase1 to select the database. You can select multiple databases to delete at the same time.
Click on Delete to complete the task.

## Manage users:
APPLIES TO:  Azure Database for MySQL - Single Server  Azure Database for MySQL - Flexible Server

This article describes how to create users for Azure Database for MySQL.

You provided a server admin username and password when creating your Azure Database for MySQL server. For more information, see this Quickstart(https://learn.microsoft.com/en-us/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal). You can determine your server admin user name in the Azure portal.
# The server admin user has these privileges:
```sql
SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER
```
After you create an Azure Database for the MySQL server, you can use the first server admin account to create more users and grant admin access to them. You can also use the server admin account to create less privileged users with access to individual database schemas.

# higlighted note in the documentation:
The SUPER privilege and DBA role aren't supported. Review the privileges(https://learn.microsoft.com/en-us/azure/mysql/concepts-limits#privileges--data-manipulation-support) in the limitations article to understand what's not supported in the service.

Password plugins like validate_password and caching_sha2_password aren't supported by the service.

# Create a database
    1. Get the connection information and admin user name.

    You need the full server name and admin sign-in credentials to connect to your database server. You can easily find the server name and sign-in information on the server Overview or the Properties page in the Azure portal.

    2. Use the admin account and password to connect to your database server. Use your preferred client tool, MySQL Workbench, mysql.exe, or HeidiSQL.

    Highlighted note: 
    If you're not sure how to connect, see connect and query data for Single Server (https://learn.microsoft.com/en-us/azure/mysql/single-server/connect-workbench) or connect and query data for Flexible Server.(https://learn.microsoft.com/en-us/azure/mysql/flexible-server/connect-workbench)

    3. Edit and run the following SQL code. Replace the placeholder value db_user with your intended new user name. Replace the placeholder value testdb with your database name.

    This SQL code creates a new database named testdb. It then makes a new user in the MySQL service and grants that user all privileges for the new database schema (testdb.*).

    ```sql
    CREATE DATABASE testdb;
    ```
# Create a nonadmin user
Now that you have created the database, you can start by creating a nonadmin user by using the CREATE USER MySQL statement.
```sql
CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';

GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';

FLUSH PRIVILEGES;
```

# Verify the user permissions
To view the privileges allowed for user db_user on testdb database, run the SHOW GRANTS MySQL statement.
```sql
USE testdb;

SHOW GRANTS FOR 'db_user'@'%';
```
# Connect to the database with the new user
Sign in to the server, specifying the designated database and using the new username and password. This example shows the MySQL command line. When you use this command, you're prompted for the user's password. Use your own server name, database name, and user name. See how to connect the single server and the flexible in the following table.

| Server type | Usage |
|-----------------|-----------------|
| Single Server | mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user@mydemoserver -p |
| Flexible Server | mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user -p |


# Limit privileges for a user
To restrict the type of operations a user can run on the database, you must explicitly add the operations in the GRANT statement. See the following example:
```sql
CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';

GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION;

FLUSH PRIVILEGES;
```

# About azure_superuser
All Azure Database for MySQL servers are created with a user called "azure_superuser". Microsoft created a system account to manage the server to conduct monitoring, backups, and other regular maintenance. On-call engineers may also use this account to access the server during an incident with certificate authentication and must request access using just-in-time (JIT) processes.


## Now we break from the documentation in Azure for documenting needed system specs

# Creation of DB:
This is taken from the txt document step_by_step_db_2.txt seen at path: teknisk/db/
# Step 1:
Install MySql
# Step 2:
```sql
CREATE DATABASE survey_db;
USE survey_db;
```
# Step 3
```sql
CREATE TABLE surveys (
  survey_id INT PRIMARY KEY AUTO_INCREMENT,
  survey_name VARCHAR(255),
  survey_description TEXT,
  start_date DATE,
  end_date DATE,
  type VARCHAR(50),
  active_flag BOOLEAN,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE survey_questions (
  question_id INT PRIMARY KEY AUTO_INCREMENT,
  survey_id INT,
  question_text TEXT,
  question_type VARCHAR(50),
  category VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (survey_id) REFERENCES surveys(survey_id)
);

CREATE TABLE survey_responses (
  response_id INT PRIMARY KEY AUTO_INCREMENT,
  survey_id INT,
  question_id INT,
  respondent_id INT,
  response_value TEXT,
  response_date TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (survey_id) REFERENCES surveys(survey_id),
  FOREIGN KEY (question_id) REFERENCES survey_questions(question_id),
  FOREIGN KEY (respondent_id) REFERENCES respondents(respondent_id)
);

CREATE TABLE respondents (
  respondent_id INT PRIMARY KEY AUTO_INCREMENT,
  survey_id INT,
  nasjonal_deleksamen_id INT,
  fs_data_id INT,
  dbh_data_id INT,
  study_program_id INT,
  institution_id INT,
  name VARCHAR(255),
  email VARCHAR(255),
  grade VARCHAR(10),
  background_info TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (survey_id) REFERENCES surveys(survey_id),
  FOREIGN KEY (nasjonal_deleksamen_id) REFERENCES nasjonal_deleksamen_results(exam_id),
  FOREIGN KEY (fs_data_id) REFERENCES fs_data(student_id),
  FOREIGN KEY (dbh_data_id) REFERENCES dbh_data(dbh_data_id),
  FOREIGN KEY (study_program_id) REFERENCES study_programs(program_id),
  FOREIGN KEY (institution_id) REFERENCES institutions(institution_id)
);

CREATE TABLE background_data_retentions (
  retention_id INT PRIMARY KEY AUTO_INCREMENT,
  respondent_id INT,
  data_type VARCHAR(50),
  retention_end_date DATE,
  deletion_status BOOLEAN,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (respondent_id) REFERENCES respondents(respondent_id)
);

CREATE TABLE nasjonal_deleksamen_results (
  exam_id INT PRIMARY KEY AUTO_INCREMENT,
  exam_subject VARCHAR(255),
  exam_date DATE,
  exam_score INT,
  national_average INT,
  institution_id INT,
  program_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(institution_id),
  FOREIGN KEY (program_id) REFERENCES study_programs(program_id)
);

#Make the table institutions
CREATE TABLE institutions (
  institution_id INT PRIMARY KEY AUTO_INCREMENT,
  institution_name VARCHAR(255),
  institution_type VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE study_programs (
  program_id INT PRIMARY KEY AUTO_INCREMENT,
  program_name VARCHAR(255),
  institution_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(institution_id)
);

CREATE TABLE audit_logs (
  log_id INT PRIMARY KEY AUTO_INCREMENT,
  action VARCHAR(255),
  entity_type VARCHAR(50),
  entity_id INT,
  action_date TIMESTAMP,
  user_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE dbh_data (
  dbh_data_id INT PRIMARY KEY AUTO_INCREMENT,
  institution_number_dbh VARCHAR(255),
  location_code_dbh VARCHAR(255),
  dbh_record TEXT,
  retrieval_date TIMESTAMP,
  source VARCHAR(255),
  program_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (program_id) REFERENCES study_programs(program_id)
);

CREATE TABLE fs_data (
  student_id INT PRIMARY KEY AUTO_INCREMENT,
  institution_number_fs VARCHAR(255),
  institution_name VARCHAR(255),
  institution_name_english VARCHAR(255),
  location_name VARCHAR(255),
  campus VARCHAR(255),
  faculty_name VARCHAR(255),
  study_program_code VARCHAR(255),
  study_program_name VARCHAR(255),
  study_program_name_english VARCHAR(255),
  study_level_code VARCHAR(10),
  study_track_code VARCHAR(10),
  study_track_name VARCHAR(255),
  language_of_instruction VARCHAR(50),
  full_time_proportion FLOAT,
  practice_percentage FLOAT,
  nus_code VARCHAR(10),
  total_students INT,
  students_in_2nd_year INT,
  students_in_5th_year INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
# Step 4:
After creating the tables, you can insert initial data using the INSERT INTO SQL command.
# Step 5:
You can now interact with the database using SQL queries. For example:

Insert data into surveys, respondents, etc.
Join tables to get meaningful data, e.g., survey responses related to a specific respondent.
