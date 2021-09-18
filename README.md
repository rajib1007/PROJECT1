# STUDENT ENQUIRY DATABASE PROJECT

An enrollment system is to help admission teams ultimately enroll more students.



# MYSQL: 

### CREATING MASTER TABLE ON MYSQL

CREATE TABLE master id int, name varchar(255) NOT NULL, email varchar(255) NOT NULL, mobile int, course varchar(255) NOT NULL, fee int, discount int, status1 varchar(255) NOT NULL, status2 varchar(255) NOT NULL);


LOAD DATA LOCAL INFILE '/home/cloudera/Desktop/STUDENT_DETAILS' INTO TABLE master
FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';


![image](https://user-images.githubusercontent.com/63140467/131074764-e333f98a-3a42-474b-b25e-ef01399d49c8.png)

### CREATING PAYMENT TABLE ON MYSQL                                

CREATE TABLE PAYMENT (id int, name varchar(255) NOT NULL, email varchar(255) NOT NULL, mobile int, course varchar(255) NOT NULL,total_fee int, paid int, ,payment_mode varchar(255) NOT NULL, installment int NOT NULL, due_date DATE);

LOAD DATA LOCAL INFILE  '/home/cloudera/Desktop/payment_table' INTO TABLE master
FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';

                    
![image](https://user-images.githubusercontent.com/63140467/131075002-5a514c1b-2d9f-418b-8209-5042dd5bbc94.png)

# SCOOP: 


### MOVING DATA FROM MYSQL TO HDFS

sqoop import --connect jdbc:mysql://192.168.150.132/RAJIB --username root --password cloudera --table master --target-dir /user/cloudera/master -m 1

### MASTER TABLE:

##### COMMANDS TO BE RUN BEFORE CREATING ORC TABLE:

##### (must run these commands aftering entering into HIVE and after entering into database i,e use databse_name;)

set hive.support.concurrency = true;
set hive.enforce.bucketing = true;
set hive.exec.dynamic.partition.mode = nonstrict;
set hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;
set hive.compactor.initiator.on = true;

### CREATING ORC MASTER TABLE:

create table master(id int, name string, email string, mobile bigint, course string, fee int, discount int, status1 string, status2 string) clustered by (id) into 4 buckets stored as orc tblproperties('transactional'='true');

##### 1.	CREATING TEMPORARY TABLE:

create table temp (id int, name string, email string, mobile bigint, course string, fee int, discount int, status1 string, status2 string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

##### 2.	LOAD DATA INTO TEMP TABLE:

load data local inpath '/home/cloudera/Desktop/demo_project/Master Table.txt' into table temp;

##### 3.	CHECK IF TEMP TABLE HAS DATA:

select * from temp;

##### 4.	TRANSFER FROM TEMP TO MASTER:

insert into table master select * from temp;

##### 5.	CHECK IF MASTER TABLE RECEIVED DATA:

select * from master;

###### (must choose Hive under Query Editor to view master ORC table)#


 ![image](https://user-images.githubusercontent.com/63140467/131075621-3a3f39f6-c3fe-44a5-8de1-c1fc59cb1d3d.png)






### JOINING TABLE:

##### 1.	CREATE JOINING TABLE:

create table joining_table(id int, name string, email string, mobile bigint, course string, joining string, day string, time string, total_fee float)row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

##### 2.	LOAD JOINING TABLE FROM MASTER TABLE QUERY:

insert overwrite table joining_table select id, name, email, mobile, course, status2 as joining, concat_ws(':',substr(status1,12,8),substr(status1,9,2),substr(status1,21,3)) as day, substr(status1,25,4) as time, ((1-(discount/100))*fee) as total_fee from master where substr(status1,1,4)=='Join';

##### 3.	CHECK IF DATA IN JOINING TABLE IS CORRECT:

select * from joining_table;

 ![image](https://user-images.githubusercontent.com/63140467/131075679-a6e1e988-2ecb-49f7-8f63-5043e4c0d47d.png)



### Joining Weekday table:

create table joining_wd(id int, name string, email string, mobile bigint, course string, joining string, day string, time string, total_fee float)row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

insert overwrite table joining_wd select id, name, email, mobile, course, status2 as joining, concat_ws(':',substr(status1,12,8),substr(status1,9,2),substr(status1,21,3)) as day, substr(status1,25,4) as time, ((1-(discount/100))*fee) as total_fee from master where substr(status1,1,4)=='Join' and substr(status1,9,2)=='WD';
 



### Joining Weekend table:

create table joining_we(id int, name string, email string, mobile bigint, course string, joining string, day string, time string, total_fee float)row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

insert overwrite table joining_we select id, name, email, mobile, course, status2 as joining, concat_ws(':',substr(status1,12,8),substr(status1,9,2),substr(status1,21,3)) as day, substr(status1,25,4) as time, ((1-(discount/100))*fee) as total_fee from master where substr(status1,1,4)=='Join' and substr(status1,9,2)=='WE';

 






### DEMO TABLE:

##### 1.	CREATE DEMO TABLE:

create table demo_table(id int, name string, email string, mobile bigint, course string, demo string, day string, time string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

##### 2.	LOAD DEMO TABLE FROM MASTER TABLE QUERY:

insert overwrite table demo_table select id, name, email, mobile, course, status2 as demo, concat_ws(':',substr(status1,9,8),substr(status1,6,2),substr(status1,18,3)) as day, substr(status1,22,4) as time from master where substr(status1,1,4)=='Demo';

##### 3.	CHECK IF DATA IN DEMO TABLE IS CORRECT:

select * from demo_table;

 
![image](https://user-images.githubusercontent.com/63140467/131075827-7f2fba2f-b500-4aaa-b49e-9ad3658cec64.png)


### Demo Weekday table:

create table demo_wd(id int, name string, email string, mobile bigint, course string, demo string, day string, time string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

insert overwrite table demo_wd select id, name, email, mobile, course, status2 as demo, concat_ws(':',substr(status1,9,8),substr(status1,6,2),substr(status1,18,3)) as day, substr(status1,22,4) as time from master where substr(status1,1,4)=='Demo' and substr(status1,6,2)=='WD';


 ![image](https://user-images.githubusercontent.com/63140467/131075747-8a8297be-1ece-4c14-a890-7c408dcdf75f.png)


### Demo Weekend table:

create table demo_we(id int, name string, email string, mobile bigint, course string, demo string, day string, time string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;

insert overwrite table demo_we select id, name, email, mobile, course, status2 as demo, concat_ws(':',substr(status1,9,8),substr(status1,6,2),substr(status1,18,3)) as day, substr(status1,22,4) as time from master where substr(status1,1,4)=='Demo' and substr(status1,6,2)=='WE';


 

![image](https://user-images.githubusercontent.com/63140467/131075768-0548f768-b4ff-44a8-8117-4801cff7f913.png)








### PAYMENT TABLE:

##### TEMPORARY PAYMENT TABLE:

create table temp_payment_table(id int, name string, email string, mobile bigint, course string,total_fee float,paid float,payment_mode String,installment int,due_date String)
row format delimited fields terminated by ',' 
lines terminated by '\n' stored as textfile;

LOAD DATA INTO TEMP PAYMENT TABLE:

load data local inpath '/home/cloudera/Desktop/payment_table' overwrite into table temp_payment_table;


##### PAYMENT ORC TABLE:

create table payment_table(id int, name string, email string, mobile bigint, course string,total_fee float,paid float,payment_mode String,installment int,due_date String) 
clustered by (id) into 2 buckets stored as orc tblproperties('transactional'='true');

INSERT DATA INTO ORC PAYMENT TABLE FROM TEMP TABLE:

insert into table payment_table select * from temp_payment_table;


![image](https://user-images.githubusercontent.com/63140467/131075989-9d769477-9bfd-49ae-a901-61bf1fc3e565.png)

 
