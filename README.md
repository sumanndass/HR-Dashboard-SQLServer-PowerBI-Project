# HR-Dashboard-SQLServer-PowerBI-Project
  - The dataset was prepared, searched, and visualized using PowerBI and SQL Server.
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
  - now we are going to add new column 'age' based on 'birthdate' column.
    ```sql
    alter table Human_Resources
    add age int;
    ```
    ```sql
    update Human_Resources
    set age = datediff(yy, birthdate, getdate());
    ```

# Data Analysis
  - 1. What is the gender breakdown of employees in the company?
    ```sql
    select gender, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by gender
    ```
  - 2. What is the race/ethnicity breakdown of employees in the company?
    ```sql
    select race, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by race
    order by no_of_emp desc
    ```
  - 3. What is the age distribution of employees in the company?
    ```sql
    with cte as
    (
    select 
    case
    	when age between 18 and 24 then '18-24'
    	when age between 25 and 34 then '25-34'
    	when age between 35 and 44 then '35-44'
    	when age between 45 and 54 then '45-54'
    	when age between 55 and 64 then '55-64'
    	else '65+'
    	end as 'age_group',
    gender
    from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    )
    select 
    age_group, gender, count(*) no_of_emp
    from cte
    group by age_group, gender
    order by age_group
    ```
  - 4. How many employees work at headquarters versus remote locations?
    ```sql
    select location, gender, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by location, gender
    order by location, gender
    ```
  - 5. What is the average length of employment for employees who have been terminated?
    ```sql
    select avg(datediff(yy, hire_date, termdate)) avg_length_employment
    from Human_Resources
    where termdate is not null -- as 'termdate' is 'resignation date'
    ```
  - 6. How does the gender distribution vary across departments and job titles?
    ```sql
    select department, gender, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by department, gender
    order by department, gender
    ```
  - 7. What is the distribution of job titles across the company?
    ```sql
    select jobtitle, gender, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by jobtitle, gender
    order by jobtitle, gender
    ```
  - 8. Which department has the highest turnover rate?
    ```sql
    with cte as
    (select department, count(*) no_of_emp,
    sum(case when termdate is not null and termdate <= getdate() then 1 else 0 end) resigned
    from Human_Resources
    group by department)
    select *, resigned*100.0/no_of_emp resigned_rate 
    from cte
    order by resigned_rate desc
    ```
  - 9. What is the distribution of employees across locations by city and state?
    ```sql
    select location_state, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by location_state
    order by location_state
    ```
    ```sql
    select location_city, count(*) no_of_emp from Human_Resources
    where termdate is null -- as 'termdate' is 'resignation date'
    group by location_city
    order by location_city
    ```
  - 10. How has the company's employee count changed over time based on hire and term dates?
    ```sql
    with cte as
    (select year(hire_date) year, count(*) hires,
    sum(case when termdate is not null and termdate <= getdate() then 1 else 0 end) resigned
    from Human_Resources
    group by year(hire_date))
    select *, hires - resigned net_change, (hires - resigned)*100.0/hires net_percent_change
    from cte
    order by year
    ```
  - 11. What is the tenure distribution for each department?
    ```sql
    select department, avg(datediff(yy, hire_date, termdate)) avg_tenure
    from Human_Resources
    where termdate is not null -- as 'termdate' is 'resignation date'
    and termdate <= getdate()
    group by department
    ```

# Data Visualization
  - Now we will vizualize the above mentioned queries in Power BI.
  - And for that reason we will connect the sql server and fetch the data using every sql query.
    - Power BI -> 'Get Data' -> 'SQL Server' -> enter 'Server' name -> enter 'Database' name -> use 'Direct Query' in 'Data Connectivity Mode' to get the table schema only -> in 'Advance' option -> enter the 'SQL Statement' to fetch required data for every query
  - And create dashboard.
    - Page 1
    ![HR_EMPLOYEE_REPORT_Page_1](https://github.com/user-attachments/assets/a9cb2627-b273-4892-b1c0-2987a5cf811d)
    - Page 2
    ![HR_EMPLOYEE_REPORT_Page_2](https://github.com/user-attachments/assets/6325a274-b543-4509-8a69-a24f1381ca26)

# An overview of the results
  - There are more male employees
  - American Indians and Native Hawaiians are the least dominant races, whereas white people are the most.
  - The youngest employee is 22 years old and the oldest is 59 years old.
  - Five age groups 18–24, 25–34, 35–44, 45–54, and 55–64 were established. Most employees were in the 35–44 age range, then the 25–34 age range, and the 18–24 age range was the lowest.
  - Instead of working remotely, many staff are based at the headquarters.
  - Employees that are fired often had worked there for 10 years.
  - Although there is a reasonably equal gender ratio across departments, there are often more males than women working there.
  - The department with the greatest turnover rate is Auditing, which is followed by Legal. The departments with the lowest turnover rates are Marketing, Business Development, and Services.
  - A large number of employees come from the state of Ohio.
  - Over time, there has been an increase in the net change in employees.
  - Every department has an average tenure of around eight years, with the greatest tenure being in Engineering, Marketing, Research and Development and Sales and the others are the lowest.

# Limitations
  - Some term dates (1599 records) were excluded from the study since they were so distant in the future. Only dates that were less than or equal to the present date were chosen.

# Reference
  - Her Data Project -- [Watch](https://www.youtube.com/watch?v=PzyZI9uLXvY)
