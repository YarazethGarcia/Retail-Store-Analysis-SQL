<img width="1212" alt="image" src="https://github.com/YarazethGarcia/Retail-Store-Analysis-SQL/assets/126752230/eee77007-68fa-49f0-ab4b-8e6910e49104">

##SCHEMA CREATION##

#Regions Table
CREATE TABLE `regions` (
  `region_id` int NOT NULL AUTO_INCREMENT,
  `region_name` varchar(40) NOT NULL,
  PRIMARY KEY (`region_id`),
  UNIQUE KEY `region_name` (`region_name`)
);

##Countries Table
CREATE TABLE `countries` 
 `country_id` int NOT NULL AUTO_INCREMENT,
  `country_code` char(2) NOT NULL,
  `country_name` varchar(40) NOT NULL,
  `region_id` int NOT NULL,
  PRIMARY KEY (`country_id`),
  UNIQUE KEY `country_code` (`country_code`),
  UNIQUE KEY `country_name` (`country_name`),
  KEY `region_id` (`region_id`),
  CONSTRAINT `countries_ibfk_1` FOREIGN KEY (`region_id`) REFERENCES `regions` (`region_id`)
);
#Location Table
CREATE TABLE `locations` (
  `location_id` int NOT NULL AUTO_INCREMENT,
  `location_code` char(4) NOT NULL,
  `street_address` varchar(40) NOT NULL,
  `postal_code` varchar(12) DEFAULT NULL,
  `city` varchar(40) NOT NULL,
  `state_province` varchar(40) DEFAULT NULL,
  `country_id` int NOT NULL,
  PRIMARY KEY (`location_id`),
  UNIQUE KEY `location_code` (`location_code`),
  KEY `country_id` (`country_id`),
  CONSTRAINT `locations_ibfk_1` FOREIGN KEY (`country_id`) REFERENCES `countries` (`country_id`)
);

##Employees Table
CREATE TABLE `employees` (
  `employee_id` int NOT NULL AUTO_INCREMENT, `employee_code` int DEFAULT NULL,
  `first_name` varchar(40) NOT NULL, `last_name` varchar(40) NOT NULL,
  `email` varchar(40) NOT NULL, `phone_number` varchar(20) DEFAULT NULL,
  `salary` double(7,2) NOT NULL, `hire_date` date DEFAULT (curdate()),
  `job_id` int NOT NULL, `location_id` int NOT NULL, `report_to` int DEFAULT NULL,
  `experience_at_VivaK` tinyint DEFAULT NULL,
  `last_performance_rating` tinyint DEFAULT NULL,
  `salary_after_increment` double(7,2) DEFAULT NULL,
  `annual_dependent_benefit` double(7,2) DEFAULT NULL,
  PRIMARY KEY (`employee_id`), UNIQUE KEY `email` (`email`),
  KEY `job_id` (`job_id`), KEY `location_id` (`location_id`),
  CONSTRAINT `employees_ibfk_1` FOREIGN KEY (`job_id`) 
  REFERENCES `organization_structure` (`job_id`),
  CONSTRAINT `employees_ibfk_2` FOREIGN KEY (`location_id`) 
  REFERENCES `locations` (`location_id`)
);

##Dependents Table
CREATE TABLE `dependents` (
  `dependent_id` int NOT NULL AUTO_INCREMENT,
  `dependent_code` int DEFAULT NULL,
  `first_name` varchar(40) NOT NULL,
  `last_name` varchar(40) NOT NULL,
  `relationship` set('Child','Spouse') NOT NULL,
  `employee_id` int NOT NULL,
  PRIMARY KEY (`dependent_id`),
  KEY `employee_id` (`employee_id`),
  CONSTRAINT `dependents_ibfk_1` FOREIGN KEY (`employee_id`) REFERENCES `employees` (`employee_id`)
);

#Organization structure Table
CREATE TABLE `organization_structure` (
  `job_id` int NOT NULL AUTO_INCREMENT,
  `job_title` varchar(40) NOT NULL,
  `min_salary` double(7,2) NOT NULL,
  `max_salary` double(7,2) NOT NULL,
  `department_name` varchar(40) NOT NULL,
  `reports_to` int DEFAULT NULL,
  `department_id` int NOT NULL,
  PRIMARY KEY (`job_id`),
  UNIQUE KEY `job_title` (`job_title`),
  KEY `department_id` (`department_id`),
  CONSTRAINT `organization_structure_ibfk_1` FOREIGN KEY (`department_id`) REFERENCES `departments` (`department_id`)
);

#Departments Table
CREATE TABLE `departments` (
  `department_id` int NOT NULL AUTO_INCREMENT,
  `department_name` varchar(40) NOT NULL,
  PRIMARY KEY (`department_id`),
  UNIQUE KEY `department_name` (`department_name`)
);


###IMPORTING AND CLEANING DATA###

#Check duplicates 
select ‘candidate keys’ 
from ‘Table’ 
group by ‘candidate keys’ 
having count(*)>1;

CREATE TABLE `dependents` (
  `dependent_id` int NOT NULL AUTO_INCREMENT,
  `dependent_code` int DEFAULT NULL,
  `first_name` varchar(40) NOT NULL,
  `last_name` varchar(40) NOT NULL,  `relationship` set('Child','Spouse') NOT NULL,
  `employee_id` int NOT NULL,
  PRIMARY KEY (`dependent_id`),
  KEY `employee_id` (`employee_id`),
  CONSTRAINT `dependents_ibfk_1` FOREIGN KEY (`employee_id`) REFERENCES `employees` (`employee_id`)
);

# Ensure the floating-point data is represented as double.
select count(*) from employees_json 
where cast(salary as unsigned) <> salary;

#Ensure that the phone numbers are all recorded in the format: ‘+000-000-000-0000
##Step 1. Added new column to store the phone number with the correct format![image]
  alter table employees_json add column phone_number_fix varchar(20);
## Step 2. Department_id in employees_json should match with the location_id. Updated  department_id according to the number of row.
  alter table hr.locations add column department_id int;
  set @rownum=0;
update hr.locations set department_id = (@rownum:=1 + @rownum);
##Step 3.  Updated phone_number_fix column. Joined ‘employees_json’ table and ‘locations’ table via 　‘department_id’.Assigning the respective country code using CASE Expression.
update vivakhr.employees_json e 
  join hr.locations l using (department_id)
  set e.phone_number_fix = concat(
  case when l.country_id in ('US','CA') then '+1'
  when l.country_id = 'UK' then '+44' 
  else '+45' end, '-', replace(e.phone_number,'.','-')
  )
  where e.phone_number is not null and e.phone_number != '';














