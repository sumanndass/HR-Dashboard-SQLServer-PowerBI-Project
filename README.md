# HR-Dashboard-SQLServer-PowerBI-Project
  - The dataset was prepared, searched, and visualized using PowerBI and SQL Server inside of a Jupyter Notebook.
  - This project is mostly focused on data cleansing and analysis using SQL Server guidance.

# Data Used
  - Data: HR Data 'Human_Resources.csv' with over 22000 rows from the year 2000 to 2020.
  - Data Cleaning & Analysis: SQL Server
  - Data Visualization: PowerBI

# Data Loading
  - first create a Database named 'Human_Resources' and use the same.
    ```sql
    create database Human_Resources;
    use Human_Resources;
    ```
  - using 'Import Data...' on right clicking on selected server you can import Human_Resources.csv dataset in the same server.
![image](https://github.com/user-attachments/assets/5571fb8c-baee-4062-a4f5-58ac43624d12)

# Data Cleaning and Transformation
  - first check the table metadata and find anything we need to modify or not in SQL Server.
    ```sql
    sp_help Human_Resources
    ```
    ![image](https://github.com/user-attachments/assets/5d0c048f-8d96-481b-a372-53d6718187b4)
  - modify the 'ï»¿id' column name to 'emp_id' and also modify the datatype 'varchar' to 'varchar(20)' of the column.
    ```sql
    EXEC sp_rename '[dbo].[Human_Resources].[ï»¿id]', 'emp_id', 'COLUMN';
    ```
    ```sql
    alter table Human_Resources
    alter column emp_id varchar(20);
    ```
  - modify the 'birthdate' column datatype 'varchar' to 'date'.
    ```sql
    alter table Human_Resources
    alter column birthdate date;
    ```
  - modify the 'hire_date' column datatype 'varchar' to 'date'.
    ```sql
    alter table Human_Resources
    alter column hire_date date;
    ```
  - modify the 'termdate' column datatype 'varchar' to 'date'.
    - but the column has many ' ' (space) values. So, we will replace them with 'null' first as we can use 'null' is date columns.
      ```sql
      update Human_Resources
      set termdate = null where termdate = ' ';
      ```
    - but the column is in 'varchar' datatype and has time and 'UTC' also. So, we will remove the time and 'UTC' as we only need the dates.
      ```sql
      update Human_Resources
      set termdate = substring(termdate, 1, 10) where termdate is not null;
      ```
    - modify the 'termdate' column datatype 'varchar' to 'date'.
      ```sql
      alter table Human_Resources
      alter column termdate date;
      ```
  - x
  - 
