Getting-started-with-SQL-Database
=================================

Getting started with SQL Database

DISCLAIMER.  Información tomada de https://www.ng.bluemix.net/docs/#services/SQLDB/index.html#sqldb_007

IBM® SQL Database for Bluemix adds a fully provisioned relational database to your application. SQL Database provides a managed database to handle the demanding web and transactional workloads of your business.

Before we can get started with SQL Database, you need to install the CloudFoundry cf tool on your desktop. Instructions on how to do that can be found in Building a web application.

You can use the Creating Apps section of the Bluemix™ documentation to create applications and services or you can do the same from command line using the CloudFoundry cf tool.
Note: These steps lay out how to use the SQL Database service using the command line interface (CLI). Most of these steps can also be done from the Bluemix user interface.

    Manage the SQL Database service instance with the cf tool.
        Login to IBM Bluemix from the command line using the following command:

        $ cf login http://api.mybluemix.net

        To see which runtimes and services are available you can use the following command:

        $ cf marketplace

        Create a new SQL Database instance with the cf create-service command. Type the following command at the terminal:

        $cf create-service sqldb sqldb_micro mySQLDB

        The first attribute is the service name, the second attribute is the plan, and the last attribute is the unique name you are giving to this service instance.
        You can use the cf services command to list all available service instances that you have created.

        $ cf services

        Before using a service in an application, you must bind the service to your application. Use the cf bind-service command, as shown below:

        $ cf bind-service myApp mySQLDB

    Connect to the SQL Database service in your application.
    After binding a SQL Database instance to the application, the following configuration is added to your VCAP_SERVICES environment variable. You can see this information in the Bluemix user interface when you go to the Runtime tab for an application to which you have bound this service.

    { sqldb": [ { "name": "SQLDB-myDB", "label": "sqldb" "plan": "sqldb_small" "credentials": { "hostname": "75.126.155.139", "host": "75.126.155.139", "port": 50000, "username": "u123456", "password": "CasDQ5v72u", "db": "I_012345", "jdbcurl": "jdbc:db2://75.126.155.139:50000/I_012345", "uri": "db2://u123456:CasDQ5v72u@75.126.155.139:50000/I_012345" } } ] }

    As you can see, the VCAP_SERVICE environment variable includes the following items:
        key: The name of the SQL Database service
        name: The name you gave to the service instance
        label: The name of the service
        plan: The selected plan
        hostname: The host name of the SQL Database server
        host: The IP address of the SQL Database server
        port: The port number of the SQL Database server
        username: The user name for authentication
        password: The password for authentication
        db: The database name
        jdbcurl: The Java™ JDBC connection string
        uri: The connection URI
    Note: There might be slight differences between the VCAP_Services shown here, the one visible through the Bluemix user interface and the one visible in the code.

    Now you can use this VCAP_Services environment variable to retrieve the information and credentials to access the SQL Database service from your application. You can do this from a variety of run-times such as Java, Node.js, or Ruby.

Accessing SQL Database from a Java application

The following code snippet shows how to obtain the service credential information and connect to the SQL Database instance using basic Java JDBC code. For an actual application, you might want to look into connection pooling, using JPA and other optimizations. This is just a sample.

The source code for this sample can be found at https://hub.jazz.net/project/pvanrun/SQLDBJava/overview in IBM DevOps Service (formerly IBM JazzHub).
Processing the VCAP_SERVICES information in Java
The code listed here imports a number of utility packages to help with the JSON parsing of the VCAP_SERVICES structure. It first checks whether this VCAP_SERVICES environment variable exists. If it does exist, it searches for a key containing the string "sqldb". If it finds this string, then it navigates to the first element of the list and gets the credentials structure. The credentials structure contains all the elements needed to access the SQL Database.

import com.ibm.nosql.json.api.*; import com.ibm.nosql.json.util.*; ... String VCAP_SERVICES = System.getenv("VCAP_SERVICES"); if (VCAP_SERVICES != null) { BasicDBObject obj = (BasicDBObject) JSON.parse(VCAP_SERVICES); String thekey = null; Set<String> keys = obj.keySet(); for (String eachkey : keys) if (eachkey.toUpperCase().contains("SQLDB")) thekey = eachkey; } } BasicDBList list = (BasicDBList) obj.get(thekey); obj = (BasicDBObject) list.get("0"); obj = (BasicDBObject) obj.get("credentials"); databaseHost = (String) obj.get("host"); databaseName = (String) obj.get("db"); port = (int) obj.get("port"); user = (String) obj.get("username"); password = (String) obj.get("password"); url = (String) obj.get("jdbcurl"); } else { // Use the jdbcurl or construct your own databaseUrl = "jdbc:db2://" + databaseHost + ":" + port + "/" + databaseName; }

Interact with the SQL Database instance to insert and query records
You can interact (insert, query, delete, update, etc.) with the SQL Database instance by using the credential information. The following example demonstrates how to connect to the database and insert a record:

try { DB2SimpleDataSource dataSource = new DB2SimpleDataSource(); dataSource.setServerName(databaseHost); dataSource.setPortNumber(port); dataSource.setDatabaseName(databaseName); dataSource.setUser(user); dataSource.setPassword (password); dataSource.setDriverType(4); con=dataSource.getConnection(); con.setAutoCommit(false); } catch (SQLException e) { // put error handling here } … Statement stmt = con.createStatement() sqlStatement = "INSERT INTO SALES.CUSTOMERS VALUES (\'John Smith\', 52)"; stmt.executeUpdate(sqlStatement);

Push your application to Bluemix
After finishing the coding, you can deploy your application to the Bluemix environment for verification. You must use the command line for this task, it is not available from the user interface. To deploy an application, you need to enter into the root directory of the Java application and use the following command:

$ cf push myApp -p myApp.war

Run your application
You are now able to use your application. You can find the application URL (the “route”) in the Bluemix user interface.
Accessing SQL Database from a Ruby Sinatra application

You can bind your Ruby Sinatra application to the SQL Database and use it to store your relational data. This section explains how to get your code running using this method. A sample is provided at the end of this section.

You can find the source code for this sample here: https://hub.jazz.net/project/pvanrun/SQLDBRuby/overview
Required components
The following components are required to connect the SQL Database to a Ruby Sinatra application. They are all described in further detail in this section.

    Gemfile
    Gemfile.lock
    config.ru
    Ruby program

Gemfile and Gemfile.lock
The gemfile contains a list of all the Ruby gems you need to include in your project. It is used by the bundler to install, update, remove and manage the gems you need. Add the following ibm_db dependency to your Ruby Gemfile:

source 'https://rubygems.org' ruby '1.9.3' gem 'sinatra', '>= 0' gem 'ibm_db', '>= 2.5.18'

Make sure bundler is installed with:

gem install bundler

Then run the following command:

bundle install

to generate your Gemfile.lock file.

For Microsoft Windows users, you can ignore a warning you will see during the cf push command later on about this Gemlock.lock file being generated on Windows. It will be auto generated. Alternatively, you can use the Gemfile and Gemfile.lock provided in the sample application.
config.ru
The config.ru file determines the starting point of your application:

require './sample' run Sinatra::Application

Connecting to SQL Database from your Ruby code

In your Ruby code parse the VCAP_SERVICES environment variable to retrieve the database connection information as shown below.

For more information on the structure of the VCAP_SERVICES environment variable see Getting started with SQL Database.

allServices = JSON.parse(ENV['VCAP_SERVICES']) sqldbServiceKey = "" allServices.keys.each { |key| # Look for the service key that matches the SQLDB service name we're looking for if key.to_s.downcase.include? “sqldb” sqldbServiceKey = key.to_s end } if sqldbServiceKey != "" sqldb = allServices[sqldbServiceKey] credentials = sqldb.first["credentials"] host = credentials["host"] username = credentials["username"] password = credentials["password"] database = credentials["db"] port = credentials["port"] dsn = "DRIVER={IBM DB2 ODBC DRIVER};DATABASE="+database+ ";HOSTNAME="+host+";PORT="+port.to_s()+";PROTOCOL=TCPIP;UID="+username+";PWD="+password+";" …

Accessing the Relational Data from the Ruby program

Now you can use the dsn datasource name to connect to the database and create a table as shown below. The variable total in the code snippet below represents the web output.

if conn = IBM_DB.connect(dsn, '', '') # To ensure the tablename is fairly unique and doesn't interfere with other users we add a timestamp # Use an explicit schema name "Bluemix" to avoid creating this table in the user's default schema. tablename = "BLUEMIX.EMPLOYEES" + Time.new.usec.to_s sql = "CREATE TABLE " + tablename + " (FIRSTNAME VARCHAR(20), LASTNAME VARCHAR(20), EMPNO INT)" if stmt = IBM_DB.exec(conn, sql) total = total + sql + "<BR><BR>\n" else out = "Statement execution failed: #{IBM_DB.stmt_errormsg}" total = total + out + "<BR>\n" end

Ruby buildpack with DB2 drivers

The base Ruby runtime within IBM Bluemix does not come with the pre-installed DB2 driver required to access the SQL Database service. Instead you can use a custom buildpack hosted at github.com. You can specify this buildpack (see below) when you push your Ruby app to the Bluemix environment, there is no need to download it separately.

https://github.com/ibmdb/db2rubybuildpack
Uploading your application
Now use the

cf push <app-name> -b https://github.com/ibmdb/db2rubybuildpack

command to upload your Ruby code, the SQL Database driver and the gem files to the Bluemix server.
Run your application
You are now able to use your application. You can find the application URL (the “route”) in the Bluemix user interface.
Accessing SQL Database from Node.js

You can bind your Node.js application to the SQL Database and use it to store your relational data. This section explains how to get your code running using this method.

The source code for this sample can be found at https://hub.jazz.net/project/pvanrun/SQLDBnode/overview in IBM DevOps Service (formerly IBM JazzHub). Node.js requires a package.json file which describes the dependencies of the application on other packages. Here is a package.json file showing the ibm_db dependency.
Package the .json file

{ "name": "SQLDBNode", "version": "0.0.1", "author": "SuperDev", "description": "A sample node app accessing SQLDB", "scripts": { "start": "node app.js" }, "dependencies": { "ibm_db" : ">=0.0.1" }, "engines": { "node": ">=0.6" } }

Process the VCAP_Services information in Node.js.
The sample code shown here searches through the VCAP_Services environment variable for a key that contains “sqldb”.

function findKey(obj,lookup) { for (var i in obj) { if (typeof(obj[i])==="object") { if (i.toUpperCase().indexOf(lookup) > -1) { // Found the key return i; } findKey(obj[i],lookup); } } return -1; } if (process.env.VCAP_SERVICES) { env = JSON.parse(process.env.VCAP_SERVICES); key = findKey(env,serviceName); }

Connect to the SQL Database service in Node.js.
Now the key found in the previous snippet is used to access the information and credentials required to access the database.

var credentials = env[key][0].credentials; var dsnString = "DRIVER={DB2};DATABASE=" + credentials.db + ";UID=" + credentials.username + ";PWD=" + credentials.password + ";HOSTNAME=" + credentials.hostname + ";port=" + credentials.port; /*Connect to the database server param 1: The DSN string which has the details of database name to connect to, user id, password, hostname, portnumber param 2: The Callback function to execute when connection attempt to the specified database is completed API for the ibm_db package can be found here: https://www.npmjs.org/package/ibm_db */ ibmdb.open(dsnString, function(err, conn) { if (err) { response.write("error: ", err.message + "<br>\n"); response.end(); }else { var sqlStatement = "SELECT NAME, CREATOR, TYPE FROM SYSIBM.SYSTABLES FETCH FIRST 50 ROWS ONLY"; /* On successful connection issue the SQL query by calling the query() function on Database param 1: The SQL query to be issued param 2: The callback function to execute when the database server responds */ conn.query(sqlStatement, function (err,tables,moreResultSets) { if (err) { response.write("SQL Error: " + err + "<br>;\n"); response.end(); } else { // process the result set

Custom buildpack with DB2 drivers

The base Node runtime within IBM Bluemix does not come with the pre-installed DB2 driver required to access the SQL Database service. Instead you can use a custom buildpack hosted at github.com. You can specify this buildpack (see below) when you push your Node app to the Bluemix environment, there is no need to download it separately.

https://github.com/ibmdb/db2nodejsbuildpack
Upload your application.
Use the following command to upload your Node.js code and the buildpack to the Bluemix server.

cf push <app-name> -b https://github.com/ibmdb/db2nodejsbuildpack

Run your application.

You are now able to use your application. You can find the application URL (the “route”) in the Bluemix user interface.
Binding SQL Database instances to Liberty applications

If you push a standalone application or a packaged Liberty server and bind to a SQL Database instance, the Liberty buildpack automatically generates or updates dataSource-related server.xml entries.
Pushing a Stand-alone Application

When you push a stand-alone application, the Liberty buildpack generates a server.xml file. If you bind to a SQL Database, the Liberty buildpack also generates the associated server.xml stanzas that are required to access the database. Additionally, the Liberty buildpack provides the required client driver JAR files. In summary, the Liberty buildpack:

    Generates a dataSource element with default properties.db2.jcc sub-element.
    Generates a jdbcDriver element.
    Generates a library element with an embedded fileset sub-element.

The Liberty buildpack must generate a jndiName in the dataSource element. Because the Liberty buildpack does not have access to the JNDI name used by the application, the Liberty buildpack generates a JNDI name with a well-known convention of "jdbc/<service_name>" where the <service_name> is the name of the bound service. For example, if you bind a SQL Database service that is named mydb to the application, the Liberty buildpack generates a jndi name of jdbc/mydb. When you are developing the application and creating the relational database service instance, the JNDI name that is used by the application must match the JNDI name that is generated by the Liberty buildpack.

The following example shows the generated configuration when an application is pushed and bound to a SQL Database service that is named mydb.

<dataSource id='db2-mydb' jdbcDriverRef='db2-driver' jndiName='jdbc/mydb' statementCacheSize='30' transactional='true'> <properties.db2.jcc databaseName='${cloud.services.mydb.connection.db}' id='db2-mydb-props' password='${cloud.services.mydb.connection.password}' portNumber='${cloud.services.mydb.connection.port}' serverName='${cloud.services.mydb.connection.host}' user='${cloud.services.mydb.connection.user}'/> </dataSource> <jdbcDriver id='db2-driver' libraryRef='db2-library'> <library id='db2-library'> <fileset dir='${server.config.dir}/lib' id='db2-fileset' includes='db2jcc4.jar db2jcc_license_cu.jar'/> </library>

Pushing a packaged server or server directory

When you push a packaged server, the Liberty buildpack detects and uses the server.xml file you provide. If you bind to one or more SQL Database service instances, you have two options:

    You can allow the Liberty buildpack to generate all necessary dataSource-related configuration for you. If you choose this option, do not include any dataSource-related configuration in your server.xml file.
    You can include the dataSource configuration in your server.xml file. If you provide mongo configuration, the buildpack will update the configuration you provide as detailed in the following sections.

If you do not need to customize the dataSource configuration, then allow the buildpack to generate the configuration for you. When the buildpack generates the configuration, it will generate default jndi names as described above. When you provide dataSource configuration in your server.xml file, you control the jndi name.

If you choose to provide the dataSource configuration in your server.xml file, then you must provide a complete and functionally correct configuration.

    You must provide dataSource, properties.db2.jcc, jdbcDriver, library and fileset elements. You must use the dir and include attributes in the fileset. Do not use the files attribute.
    The featureManager element must contain the jdbc-4.0 feature.
    You can optionally provide db2 client access jars. If you do not provide them, the buildpack will provide them for you.

If you provide client driver jar files, place them in the wlp/usr/servers/<servername>/lib directory. Your client jars must use the original names when you obtained them.
User-provided dataSource configuration

If you provide dataSource configuration in the server.xml file, the Liberty buildpack must be able to find the existing configuration to update it.

    If you bind to a single relational database instance and provide a single dataSource configuration, there is a 1-1 mapping between instance and configuration.
    If you bind to multiple relational database instances (and provide the requisite configuration for each) then you need to provide well-known configuration ids to allow the buildpack to map a service instance to a configuration instance.

Well-known configuration ids must use the following patterns:

    The dataSource element must use a configuration id of db2-<service_name>.
    The properties.db2.jcc element must use a configuration id of db2-<service_name>-props.
    The jdbcDriver element must use a configuration id of db2-driver.
    The library element must use a configuration id of db2-library.
    The fileset element must use a configuration id of db2-fileset.

Note: In this case, the Liberty buildpack does not update the jndiName.

The following code is a sample of dataSource configuration that might be provided in the local server.xml file. This sample assumes that when the packaged server is pushed to the cloud, it is bound to a SQLDB database named "mydb". In this case, the jndiName can be whatever value you prefer. The jndiName is independent of the service name.

<dataSource id='db2-mydb' jdbcDriverRef='db2-driver' jndiName='MyDataSource' statementCacheSize='30' transactional='true'> <properties.db2.jcc id='db2-mydb-props' databaseName='test' user='someuser' password='passw0rd' portNumber='3306' serverName='localhost'> </dataSource> <jdbcDriver id='db2-driver' libraryRef='db2-library'> <library id='db2-library'> <fileset id='db2-fileset' dir='C:/db2drivers' includes='db2jcc4.jar db2jcc_license_cu.jar'> </library>

SQL Database Console

The SQL Database console can be accessed from the Bluemix user interface by clicking on the SQL Database instance.

It provides the following functions:

    Create and drop objects. View object details, browse data and export data.
    Load the data from your data source into a database table for the cloud.
    Back up the data for the cloud database to store it in a different location.
    Restore the data to the cloud database from a different location.
    View the applications that are connected to a database.
    View data privacy reports generated by IBM Guardium that show objects which contain sensitive data or activity which occurs on sensitive data.
    View SQL statements for a database.
    View the list of table spaces and their status in a database.

SQL Database Sensitive Data Reporting, powered by IBM InfoSphere Guardium

SQL Database currently comes with a reporting feature, powered by IBM InfoSphere Guardium, which allows for the detection and monitoring of sensitive objects in a SQL Database. This feature is available by default. It consists of a series of reports that identify sensitive data, such as credit cards and social security numbers, and provides reports about connections and activities accessing those objects. These reports can be used to provide access information required for PCI/DSS audits.

Today’s IT environments are greatly concerned about the handling of sensitive data, especially personally identifiable information (PII), because it represents risk and possible liability in the case of disclosure or mishandling. Many times this sensitive data is processed in databases without the knowledge of the organization. The SQL Database service includes seamless functionality to not only discover the presence of sensitive data, but also to monitor the access to that data. This non-intrusive capability helps the database users to have better awareness about the handling of their sensitive data and to take appropriate measures to protect it.

The capabilities provided by these reports relate to a particular instance of the SQL Database service, but they can be expanded to a enterprise data security and compliance deployment using the full InfoSphere Guardium implementation. A full InfoSphere Guardium implementation can provide you with cross-enterprise real-time monitoring and auditing of all data sources, with alerting, blocking and masking capabilities; which are necessary for a complete data security and compliance strategy.
Accessing Reports on Sensitive Data
The reports are available in the Managed Database Console in the Data Privacy section. To get to the Console, click on the service instance in the IBM Bluemix user interface.
The reporting process consists of two steps

    Identification of Sensitive Objects

    In this step the Sensitive Data feature scans your databases looking for patterns of data that would indicate sensitive elements, such as credit cards or social security numbers. The discovered data is not stored or replicated anywhere, tables and columns in question are just marked as potentially containing data that should be appropriately secured. The process to identify these sensitive objects runs on a scheduled basis (every 8 to 10 hours). There is a report available in the SQL Database console in the Data Privacy section, showing the objects in questions. One option to prevent access to this data would be to the use SQL Database Data Masking feature (powered by InfoSphere Optim) documented in SQL Database Data Privacy Feature, powered by Optim.
    Reporting on access to those objects

    Once the objects are discovered they are added to a scan list and access to them is monitored by the Sensitive Data feature. Reports on this access can be retrieved from the same section in the console and the details of the reports are described below.

Below are the descriptions of the four reports used for the SQL Database Sensitive Data Discovery and Monitoring feature, powered by IBM InfoSphere Guardium. The report definitions cannot be changed, nor can additional reports be added.
SQL Database Discovered Sensitive Objects
This report retrieves the sensitive objects found by the classification process in a specified SQL Database. These are the objects containing sensitive data such as credit card numbers or Social Security numbers. For each sensitive object it displays:

    Database name
    Schema
    Table Name (View, synonym etc.)
    Column Name (Column containing the sensitive data)

SQL Database Connection Profile
This report displays the connections made to the specified database during a given period of time. The report displays:

    Max Session Start: When the last connection happened for the client IP, server IP, source program, database User name and database name
    Database Name
    Client IP
    Server IP
    Database User Name
    Source Program
    Count of Sessions to the specified database for the client IP, server IP, program, user and database names.

SQL Database Sensitive Objects Activity Summary
Provides a summary on the activity on the sensitive objects being monitored on the specified database during a given period of time. The report displays:

    Client IP: Client IP from which the sensitive object was accessed
    DB User Name:
    Database Name
    SQL Verb: SQL command used to access the object (Select, insert, alter Table, Update, etc.)
    Object Name: Name of the sensitive object accessed
    Total Access: total number of times the object was accessed by the Db user using the specific Verb from the same client IP during the given period of time.

Note: Due to the nature of the hosting environment this report should be considered as a good approximation of the access patterns on the sensitive data, and not as an audit record with complete view of the activity.
SQL Database Sensitive Objects Detailed Activity
This report provides details on the activities on the sensitive objects on a specified database during a specific period of time, the report displays:

    Client Ip: IP of the client from which the sensitive object was accessed
    Database Name
    DB Username
    Timestamp: When the specific command happened
    Full SQL: SQL used to access the sensitive object (the complete SQL statement)

Note: Due to the nature of the hosting environment this report should be considered as a good approximation of the access patterns on the sensitive data, and not as an audit record with complete view of the activity.
Expanding Security and compliance coverage
If the information provided by the SQL Database sensitive data reports is helpful to have confidence on the handling of sensitive data, there is more extensive data security, compliance, and privacy materials available for exploration at the following link: http://www.ibm.com/software/data/guardium/
SQL Database Data Privacy Feature, powered by Optim

SQL Database currently offers extended functionality by providing access to the Optim Data Privacy Provider Library. This set of User Defined Functions (UDFs) can be used to mask data that might be sensitive in nature and replace it with fake values that still adhere to data validity checks. The Optim data privacy provider library includes functions that can mask specific data such as credit card or national ID numbers and can mask data when a contextually accurate, yet fictionalized, value is appropriate.

The library includes providers that mask specific data such as credit card numbers, date/time values, email addresses, and national ID numbers.
The library also includes providers that feature various masking functions:

    Use the affinity privacy provider to mask data while maintaining the format and character types of the source values.
    Use the hash privacy provider to mask source data with numeric values generated by a hash algorithm.

For more information, refer to Optim data privacy provider library.
Use within an IBM Bluemix application
The Optim UDF functions are available by default in all SQL Database instances within Bluemix. They can be used with SQL commands issued from Bluemix applications. They have been deployed in the

DB2INST1 Schema

As the following UDFs:

    DB2INST1.OPTIMMASKDATE
    DB2INST1.OPTIMMASKDOUBLE
    DB2INST1.OPTIMMASKINT64
    DB2INST1.OPTIMMASKSTR
    DB2INST1.OPTIMMASKSTR2
    DB2INST1.OPTIMMASKSTR2INT
    DB2INST1.OPTIMMASKSTR3
    DB2INST1.OPTIMMASKSTR4
    DB2INST1.OPTIMMASKSTR5
    DB2INST1.OPTIMMASKSTR6
    DB2INST1.OPTIMMASKSTRINT
    DB2INST1.OPTIMMASKSTRUFT16
    DB2INST1.OPTIMMASKTIMESTAMP

Some examples are shown below. There are several ways in which these UDF functions can be used in the context of a Bluemix application:

    Using CREATE TABLE AS SELECT
    Using In-table update
    Using a VIEW
    Using a SELECT clause
    Using Fine Grained Access Control (FGAC)

Using CREATE TABLE AS SELECT
In this option we copy the structure and the content from the original table into a new table in which we store only the masked data. The use case here could be that we need test data for QA and want to use realistic data without exposing our production data to our QA team. The first step is to create a copy of the original table using a CREATE TABLE AS SELECT statement. In the second step we use an insert to moved the masked data into the second table. In the sample below every credit card number gets replaced by a fake, but realistic alternative number. For the sake of this example we only masked the credit card number, obviously other data elements, such as the CVV number, should be masked as well to make the data more secure.

CREATE TABLE USER01.OPTIM_CUSTOMERS_MASKED AS (SELECT * FROM USER01.OPTIM_CUSTOMERS) WITH NO DATA INSERT INTO USER01.OPTIM_CUSTOMERS_MASKED ( CUST_ID,CUSTNAME,ADDRESS1,ADDRESS2,LOCALITY,CITY,STATE, COUNTRY_CODE,POSTAL_CODE,POSTAL_CODE_PLUS4,EMAIL_ADDRESS, PHONE_NUMBER,YTD_SALES,SALESMAN_ID,NATIONALITY,NATIONAL_ID, CREDITCARD_NUMBER, CREDITCARD_TYPE,CREDITCARD_EXP,CREDITCARD_CVV, DRIVER_LICENSE,CREDITCARD_HISTORY ) SELECT CUST_ID,CUSTNAME,ADDRESS1,ADDRESS2,LOCALITY,CITY,STATE, COUNTRY_CODE,POSTAL_CODE,POSTAL_CODE_PLUS4,EMAIL_ADDRESS, PHONE_NUMBER,YTD_SALES,SALESMAN_ID,NATIONALITY,NATIONAL_ID, DB2INST1.OptimMaskStr(CREDITCARD_NUMBER,'pro=ccn,method=repeatable,pattern=6C,wheninvalid=preserve,flddef1=(name=c1,dt=varchar)'), CREDITCARD_TYPE,CREDITCARD_EXP,CREDITCARD_CVV, DRIVER_LICENSE,CREDITCARD_HISTORY FROM USER01.OPTIM_CUSTOMERS

Using In-table update
In this example the data is updated in place, the original values in the column are replaced by masked versions. Use this method if the original values are no longer required in this table.

UPDATE USER01.OPTIM_CUSTOMERS SET CREDITCARDNUMBER = DB2INST1.OptimMaskStr(CREDITCARD_NUMBER,'pro=ccn,method=repeatable,pattern=6C,wheninvalid=preserve,flddef1=(name=c1,dt=varchar)')

Using a VIEW
This method creates a view on the original table using the Optim masking functions. The original data is maintained in the table. To use this in practice, access to the original table would be restricted through use of database rights and privileges and only the masked view would be available for common usage.

CREATE VIEW USER01.OPTIM_CUSTOMERS_VIEW ( CUST_ID,CUSTNAME,ADDRESS1,ADDRESS2,LOCALITY,CITY,STATE, COUNTRY_CODE,POSTAL_CODE,POSTAL_CODE_PLUS4,EMAIL_ADDRESS, PHONE_NUMBER,YTD_SALES,SALESMAN_ID,NATIONALITY,NATIONAL_ID, CREDITCARD_NUMBER, CREDITCARD_TYPE,CREDITCARD_EXP,CREDITCARD_CVV, DRIVER_LICENSE,CREDITCARD_HISTORY ) AS SELECT CUST_ID,CUSTNAME,ADDRESS1,ADDRESS2,LOCALITY,CITY,STATE, COUNTRY_CODE,POSTAL_CODE,POSTAL_CODE_PLUS4,EMAIL_ADDRESS, PHONE_NUMBER,YTD_SALES,SALESMAN_ID,NATIONALITY,NATIONAL_ID, DB2INST1.OptimMaskStr(CREDITCARD_NUMBER,'pro=ccn,method=repeatable,pattern=6C,wheninvalid=preserve,flddef1=(name=c1,dt=varchar)'), CREDITCARD_TYPE,CREDITCARD_EXP,CREDITCARD_CVV, DRIVER_LICENSE,CREDITCARD_HISTORY FROM USER01.OPTIM_CUSTOMERS

Using a SELECT clause
Here the data is masked every time it is accessed by using the masking function inside of the SQL SELECT clause. Obviously this is a less secure method of protecting the data as it is still available to the issuer of the SQL in its original form in the table.

SELECT CUST_ID,CUSTNAME,ADDRESS1,ADDRESS2,LOCALITY,CITY,STATE, COUNTRY_CODE,POSTAL_CODE,POSTAL_CODE_PLUS4,EMAIL_ADDRESS, PHONE_NUMBER,YTD_SALES,SALESMAN_ID,NATIONALITY,NATIONAL_ID, DB2INST1.OptimMaskStr(CREDITCARD_NUMBER,'pro=ccn,method=repeatable,pattern=6C,wheninvalid=preserve,flddef1=(name=c1,dt=varchar)'), CREDITCARD_TYPE,CREDITCARD_EXP,CREDITCARD_CVV, DRIVER_LICENSE,CREDITCARD_HISTORY FROM USER01.OPTIM_CUSTOMERS

Using Fine Grained Access Control (FGAC)
Here SQL Database's Fine Grained Access Control is used to selectively mask information depending on the permissions of the database user. Specific permissions are created for different authorization IDs, allowing certain users to see the original business data while others see the masked data.

Imagine a database containing sensitive information is accessed by both business users who are entitled to see the actual data and QA staff who isn't. To accommodate this scenario, two permissions can be created along with a mask to selectively return the original business data or the masked version.
Create business user permission:

CREATE PERMISSION BUS_ROW_ACCESS ON EMPLOYEE FOR ROWS WHERE VERIFY_GROUP_FOR_USER(SESSION_USER,'BUS') = 1 ENFORCED FOR ALL ACCESS ENABLE;

Create QA user permission:

CREATE PERMISSION QA_ROW_ACCESS ON EMPLOYEE FOR ROWS WHERE VERIFY_GROUP_FOR_USER(SESSION_USER,'QA') = 1 ENFORCED FOR ALL ACCESS ENABLE; COMMIT;

Activate row access control:

ALTER TABLE EMPLOYEE ACTIVATE ROW ACCESS CONTROL; COMMIT;

Create mask:

CREATE MASK SSN_MASK ON EMPLOYEE FOR COLUMN SSN RETURN CASE WHEN (VERIFY_GROUP_FOR_USER(SSESSION_USER,'BUS') = 1) THEN SSN WHEN (VERIFY_GROUP_FOR_USER(SESSION_USER,'QA') = 1) THEN OptimMaskStr(SSN, 'PRO=NID,SWITCH=US,FLDDEF1=(NAME=SSN,DT=CHAR)') ELSE NULL END ENABLE; COMMIT;

Activate column access control:

ALTER TABLE EMPLOYEE ACTIVATE COLUMN ACCESS CONTROL; COMMIT;

Known restrictions

SQL Database offers a cloud-based instance of IBM DB2 Enterprise Server Edition V10.5. For more information on using this service, refer to the online documentation in the IBM DB2 v10.5 Knowledge Center.

Note that because of its nature as a cloud-based offering, there are several restrictions and limitations on the DB2 database provided with this SQL Database service and parts of the online documentation do not apply here.
