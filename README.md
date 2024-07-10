Raw Data
=================================================================================================================================================================
optimize this logic into an algorithm that can be used to write T-SQL code or python code to be used to resolve this project challenges [Prompt:
I have an MPI_DataWarehouse with about 5000 tables that are extracted from an application called Rejmic_system, I want to identify all the unused users(please note that user(s) are considered anyone with a role that allows them to log into  Rejmic_system.

Below is the logic/Algorithm  I am thinking of using to achieve this task, please optimise wherever you think will make the task easy to achive  :

 1. Check all the columns in the tables stored in MPI_DataWarehouse identify users that are not marked as inactive or logically deleted  and create a table to store extracted tables with these columns
MPI_Table1 [
    Username NVARCHAR(255),
    FamilyName NVARCHAR(255),
    FirstName NVARCHAR(255),
    UsernameID NVARCHAR(255),
    LastLogin DATETIME,
    LastSeen DATETIME,
    Estimate NVARCHAR(255),
    Specialty NVARCHAR(255),
    SystemID NVARCHAR(255),
    Band NVARCHAR(255),
    User_Type NVARCHAR(255),
    EmailAddress NVARCHAR(255)
]
2. For each user in the list cycle through and identify their date of last login and their specialty by checking all the columns in the  tables in MPI_DataWarehouse that have login information and populate the field LastLogin and specialty with the value extracted or Null if the data is not available.
3. Next... For each user where LastLogin is missing (Null) cycle through and identify the date they last saw a patient/specialty from the Audit Trail and populate the field LastSeen / specialty with value.
4. Write a logic on how to update the  User_type and where the User type  remains "Null" The assumption is these are external users and therefore Type should be populated with "External".
5. For the band column derives the Band column based on the current system data - (LastLogon/Lastseen), if the band Month is less than 12 months update the column as Assumed Current  , if the month is equal to 12 or greater than 12 month Assumed to be unused ]

Optimzed Data 
========================================================================================================================================================================
#### Step-by-Step Algorithm:

1. **Create a Permanent Table to Store Extracted User Data:**
    - Create a permanent table `MPI_ExtractedUsers` in the `MPI_DataWarehouse` to store user information extracted from relevant tables.

2. **Identify Relevant Tables and Columns:**
    - Dynamically identify all relevant columns across the 5000 tables that might contain user-related information such as `Username`, `FamilyName`, `FirstName`, etc.
    - Prompt: {Write a T-SQL code that Identifies Relevant Tables and Columns in MPI_DataWarehouse and Dynamically identify all relevant columns across the 5000 tables that might contain user-related information such as Username, FamilyName, FirstName, UsernameID, LastLogin, LastSeen, Estimate, Specialty, SystemID, Band, User_Type and  EmailAddress.}
    - Result from Prompt
  ======================================================
    - -- Define the list of relevant column names
DECLARE @relevantColumns TABLE (ColumnName NVARCHAR(100));
INSERT INTO @relevantColumns (ColumnName)
VALUES ('Username'), ('FamilyName'), ('FirstName'), ('UsernameID'), ('LastLogin'),
       ('LastSeen'), ('Estimate'), ('Specialty'), ('SystemID'), ('Band'), ('User_Type'),
       ('EmailAddress');

-- Variable to store dynamic SQL query
DECLARE @sql NVARCHAR(MAX);

-- Query to identify tables and columns
SET @sql = N'';

SELECT @sql = @sql + 
    N'SELECT TableName = ''' + t.name + N''', ColumnName = ''' + c.name + N''', DataType = ''' + 
    ty.name + N'''
    FROM ' + QUOTENAME(s.name) + N'.' + QUOTENAME(t.name) + N' AS ' + QUOTENAME(c.name) + N'
    WHERE ' + QUOTENAME(c.name) + N' IN (''' + STRING_AGG(rc.ColumnName, N''', ''') WITHIN GROUP (ORDER BY rc.ColumnName) + N''')
    UNION ALL '
FROM sys.schemas s
JOIN sys.tables t ON s.schema_id = t.schema_id
JOIN sys.columns c ON t.object_id = c.object_id
JOIN sys.types ty ON c.system_type_id = ty.system_type_id
JOIN @relevantColumns rc ON c.name = rc.ColumnName
WHERE s.name = 'Clinical_Systems' -- Specify your schema name here
  AND t.is_ms_shipped = 0 -- Exclude system tables
GROUP BY s.name, t.name, c.name, ty.name; -- Group by all selected columns to satisfy the requirement

-- Remove the last 'UNION ALL ' if @sql is not empty
IF LEN(@sql) > 0
BEGIN
    -- Remove the last 'UNION ALL '
    SET @sql = LEFT(@sql, LEN(@sql) - LEN(' UNION ALL '));

    -- Execute dynamic SQL
    EXEC sp_executesql @sql;
END
ELSE
BEGIN
    PRINT 'Dynamic SQL is empty. No valid query to execute.';
END
===================================================================================================================================

3. **Extract User Data:**
    - Extract user data from identified columns and populate the `MPI_ExtractedUsers` table.

4. **Update `LastLogin` and `Specialty` Fields:**
    - Update the `LastLogin` and `Specialty` fields by checking all columns in the tables that contain login information.

5. **Update `LastSeen` Field:**
    - For users with `LastLogin` missing (NULL), update the `LastSeen` field by checking the audit trail or relevant tables.

6. **Update `User_Type` Field:**
    - Populate the `User_Type` field, setting it to "External" for users where `User_Type` remains NULL.

7. **Update `Band` Field:**
    - Derive the `Band` column based on `LastLogin` or `LastSeen`. If the last activity is less than 12 months, set the `Band` as "Assumed Current"; otherwise, set it as "Assumed to be unused".

#### T-SQL Implementation:

1. **Create the Permanent Table:**
    ```sql
    USE MPI_DataWarehouse;
    GO
    
    IF OBJECT_ID('dbo.MPI_ExtractedUsers', 'U') IS NOT NULL
        DROP TABLE dbo.MPI_ExtractedUsers;
    
    CREATE TABLE dbo.MPI_ExtractedUsers (
        Username NVARCHAR(255),
        FamilyName NVARCHAR(255),
        FirstName NVARCHAR(255),
        UsernameID NVARCHAR(255),
        LastLogin DATETIME,
        LastSeen DATETIME,
        Estimate NVARCHAR(255),
        Specialty NVARCHAR(255),
        SystemID NVARCHAR(255),
        Band NVARCHAR(255),
        User_Type NVARCHAR(255),
        EmailAddress NVARCHAR(255)
    );
    ```

2. **Identify and Extract User Data:**
    ```sql
    DECLARE @sql NVARCHAR(MAX) = N'';
    DECLARE @tableName NVARCHAR(255);
    DECLARE @columnName NVARCHAR(255);

    -- Cursor to find user-related columns
    DECLARE table_cursor CURSOR FOR
    SELECT DISTINCT TABLE_NAME, COLUMN_NAME
    FROM MPI_DataWarehouse.INFORMATION_SCHEMA.COLUMNS
    WHERE COLUMN_NAME LIKE '%Username%' OR COLUMN_NAME LIKE '%FamilyName%' OR COLUMN_NAME LIKE '%FirstName%'
       OR COLUMN_NAME LIKE '%UsernameID%' OR COLUMN_NAME LIKE '%LastLogin%' OR COLUMN_NAME LIKE '%LastSeen%'
       OR COLUMN_NAME LIKE '%Estimate%' OR COLUMN_NAME LIKE '%Specialty%' OR COLUMN_NAME LIKE '%SystemID%'
       OR COLUMN_NAME LIKE '%Band%' OR COLUMN_NAME LIKE '%User_Type%' OR COLUMN_NAME LIKE '%EmailAddress%';

    OPEN table_cursor;
    FETCH NEXT FROM table_cursor INTO @tableName, @columnName;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @sql += 'INSERT INTO dbo.MPI_ExtractedUsers (Username, FamilyName, FirstName, UsernameID, LastLogin, LastSeen, Estimate, Specialty, SystemID, Band, User_Type, EmailAddress)
                     SELECT [' + @columnName + '], ''FamilyName'', ''FirstName'', ''UsernameID'', ''LastLogin'', ''LastSeen'', ''Estimate'', ''Specialty'', ''SystemID'', ''Band'', ''User_Type'', ''EmailAddress''
                     FROM ' + @tableName + ' WHERE [' + @columnName + '] IS NOT NULL
                     UNION ALL ';

        FETCH NEXT FROM table_cursor INTO @tableName, @columnName;
    END

    CLOSE table_cursor;
    DEALLOCATE table_cursor;

    -- Remove the trailing 'UNION ALL '
    SET @sql = LEFT(@sql, LEN(@sql) - LEN('UNION ALL '));

    -- Execute the dynamic SQL to populate the permanent table
    EXEC sp_executesql @sql;
    ```

3. **Update `LastLogin` and `Specialty` Fields:**
    ```sql
    -- Assuming there are tables containing login information such as UserLogins
    UPDATE dbo.MPI_ExtractedUsers
    SET LastLogin = ul.LastLogin,
        Specialty = ul.Specialty
    FROM dbo.MPI_ExtractedUsers mpi
    INNER JOIN (
        SELECT Username, MAX(LoginDate) AS LastLogin, Specialty
        FROM UserLogins
        GROUP BY Username, Specialty
    ) ul ON mpi.Username = ul.Username
    WHERE mpi.LastLogin IS NULL;
    ```

4. **Update `LastSeen` Field:**
    ```sql
    -- Update LastSeen if LastLogin is NULL
    UPDATE dbo.MPI_ExtractedUsers
    SET LastSeen = au.LastSeen
    FROM dbo.MPI_ExtractedUsers mpi
    INNER JOIN (
        SELECT Username, MAX(AuditDate) AS LastSeen
        FROM AuditTrail
        GROUP BY Username
    ) au ON mpi.Username = au.Username
    WHERE mpi.LastLogin IS NULL AND mpi.LastSeen IS NULL;
    ```

5. **Update `User_Type` Field:**
    ```sql
    UPDATE dbo.MPI_ExtractedUsers
    SET User_Type = ISNULL(User_Type, 'External')
    WHERE User_Type IS NULL;
    ```

6. **Update `Band` Field:**
    ```sql
    UPDATE dbo.MPI_ExtractedUsers
    SET Band = CASE 
                   WHEN DATEDIFF(MONTH, ISNULL(LastLogin, LastSeen), GETDATE()) < 12 THEN 'Assumed Current'
                   ELSE 'Assumed to be unused'
               END;
    ```

#### Python Implementation:

1. **Create the Permanent Table (if not already created):**
    ```python
    import pyodbc

    connection = pyodbc.connect('DRIVER={SQL Server};SERVER=server_name;DATABASE=MPI_DataWarehouse;Trusted_Connection=yes;')
    cursor = connection.cursor()

    create_table_query = '''
    IF OBJECT_ID('dbo.MPI_ExtractedUsers', 'U') IS NOT NULL
        DROP TABLE dbo.MPI_ExtractedUsers;
    
    CREATE TABLE dbo.MPI_ExtractedUsers (
        Username NVARCHAR(255),
        FamilyName NVARCHAR(255),
        FirstName NVARCHAR(255),
        UsernameID NVARCHAR(255),
        LastLogin DATETIME,
        LastSeen DATETIME,
        Estimate NVARCHAR(255),
        Specialty NVARCHAR(255),
        SystemID NVARCHAR(255),
        Band NVARCHAR(255),
        User_Type NVARCHAR(255),
        EmailAddress NVARCHAR(255)
    );
    '''
    cursor.execute(create_table_query)
    connection.commit()
    ```

2. **Identify and Extract User Data:**
    ```python
    extract_user_data_query = '''
    DECLARE @sql NVARCHAR(MAX) = N'';
    DECLARE @tableName NVARCHAR(255);
    DECLARE @columnName NVARCHAR(255);

    DECLARE table_cursor CURSOR FOR
    SELECT DISTINCT TABLE_NAME, COLUMN_NAME
    FROM MPI_DataWarehouse.INFORMATION_SCHEMA.COLUMNS
    WHERE COLUMN_NAME LIKE '%Username%' OR COLUMN_NAME LIKE '%FamilyName%' OR COLUMN_NAME LIKE '%FirstName%'
       OR COLUMN_NAME LIKE '%UsernameID%' OR COLUMN_NAME LIKE '%LastLogin%' OR COLUMN_NAME LIKE '%LastSeen%'
       OR COLUMN_NAME LIKE '%Estimate%' OR COLUMN_NAME LIKE '%Specialty%' OR COLUMN_NAME LIKE '%SystemID%'
       OR COLUMN_NAME LIKE '%Band%' OR COLUMN_NAME LIKE '%User_Type%' OR COLUMN_NAME LIKE '%EmailAddress%';

    OPEN table_cursor;
    FETCH NEXT FROM table_cursor INTO @tableName, @columnName;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @sql += 'INSERT INTO dbo.MPI_ExtractedUsers (Username, FamilyName, FirstName, UsernameID, LastLogin, LastSeen, Estimate, Specialty, SystemID, Band, User_Type, EmailAddress)
                     SELECT [' + @columnName + '], ''FamilyName'', ''FirstName'', ''UsernameID'', ''LastLogin'', ''LastSeen'', ''Estimate'', ''Specialty'', ''SystemID'', ''Band'', ''User_Type'', ''EmailAddress''
                     FROM ' + @tableName + ' WHERE [' + @columnName + '] IS NOT NULL
                     UNION ALL ';

        FETCH NEXT FROM table_cursor INTO @tableName, @columnName;
    END

    CLOSE table_cursor;
    DEALLOCATE table_cursor;

    SET @sql = LEFT(@sql, LEN(@sql) - LEN('UNION ALL '));

    EXEC sp_executesql @sql;
    '''

    cursor.execute(extract_user_data_query)
    connection.commit()
    ```

3. **Update `LastLogin` and `Specialty` Fields:**
    ```python
    update_lastlogin_specialty_query = '''
    UPDATE dbo.MPI_ExtractedUsers
    SET LastLogin = ul.LastLogin,
        Specialty = ul.Specialty
    FROM dbo.MPI_ExtractedUsers mpi
    
    INNER JOIN (
        SELECT Username, MAX(LoginDate) AS LastLogin, Specialty


    ==========================================================================================================
USE Clinical_Systems;
GO

IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'MPI_ExtractedUsers')
BEGIN
    CREATE TABLE dbo.MPI_ExtractedUsers (
        Username NVARCHAR(255),
        FamilyName NVARCHAR(255),
        FirstName NVARCHAR(255),
        UsernameID NVARCHAR(255),
        LastLogin DATETIME,
        LastSeen DATETIME,
        Estimate NVARCHAR(255),
        Specialty NVARCHAR(255),
        SystemID NVARCHAR(255),
        Band NVARCHAR(255),
        User_Type NVARCHAR(255),
        EmailAddress NVARCHAR(255)
    );
END
GO






    DECLARE @sql NVARCHAR(MAX);
DECLARE @tableName NVARCHAR(255);
DECLARE @columnName NVARCHAR(255);

-- Ensure no cursor with the same name exists
IF CURSOR_STATUS('global', 'table_cursor') >= -1
BEGIN
    DEALLOCATE table_cursor;
END

-- Cursor to find user-related columns
DECLARE table_cursor CURSOR FOR
SELECT DISTINCT TABLE_NAME, COLUMN_NAME
FROM Clinical_Systems.INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME LIKE '%Username%' OR COLUMN_NAME LIKE '%FamilyName%' OR COLUMN_NAME LIKE '%FirstName%'
   OR COLUMN_NAME LIKE '%UsernameID%' OR COLUMN_NAME LIKE '%LastLogin%' OR COLUMN_NAME LIKE '%LastSeen%'
   OR COLUMN_NAME LIKE '%Estimate%' OR COLUMN_NAME LIKE '%Specialty%' OR COLUMN_NAME LIKE '%SystemID%'
   OR COLUMN_NAME LIKE '%Band%' OR COLUMN_NAME LIKE '%User_Type%' OR COLUMN_NAME LIKE '%EmailAddress%';

OPEN table_cursor;
FETCH NEXT FROM table_cursor INTO @tableName, @columnName;


SET @sql = N'';

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql += 'INSERT INTO dbo.MPI_ExtractedUsers (Username, FamilyName, FirstName, UsernameID, LastLogin, LastSeen, Estimate, Specialty, SystemID, Band, User_Type, EmailAddress)
                 SELECT 
                     CASE WHEN COLUMN_NAME = ''Username'' THEN ' + @columnName + ' END AS Username,
                     CASE WHEN COLUMN_NAME = ''FamilyName'' THEN ' + @columnName + ' END AS FamilyName,
                     CASE WHEN COLUMN_NAME = ''FirstName'' THEN ' + @columnName + ' END AS FirstName,
                     CASE WHEN COLUMN_NAME = ''UsernameID'' THEN ' + @columnName + ' END AS UsernameID,
                     CASE WHEN COLUMN_NAME = ''LastLogin'' THEN ' + @columnName + ' END AS LastLogin,
                     CASE WHEN COLUMN_NAME = ''LastSeen'' THEN ' + @columnName + ' END AS LastSeen,
                     CASE WHEN COLUMN_NAME = ''Estimate'' THEN ' + @columnName + ' END AS Estimate,
                     CASE WHEN COLUMN_NAME = ''Specialty'' THEN ' + @columnName + ' END AS Specialty,
                     CASE WHEN COLUMN_NAME = ''SystemID'' THEN ' + @columnName + ' END AS SystemID,
                     CASE WHEN COLUMN_NAME = ''Band'' THEN ' + @columnName + ' END AS Band,
                     CASE WHEN COLUMN_NAME = ''User_Type'' THEN ' + @columnName + ' END AS User_Type,
                     CASE WHEN COLUMN_NAME = ''EmailAddress'' THEN ' + @columnName + ' END AS EmailAddress
                 FROM ' + @tableName + ' 
                 WHERE ' + @columnName + ' IS NOT NULL
                 UNION ALL ';

    FETCH NEXT FROM table_cursor INTO @tableName, @columnName;
END

CLOSE table_cursor;
DEALLOCATE table_cursor;

-- Remove the trailing 'UNION ALL '
IF LEN(@sql) > 0
BEGIN
    SET @sql = LEFT(@sql, LEN(@sql) - LEN('UNION ALL '));
END

-- Execute the dynamic SQL
IF LEN(@sql) > 0
BEGIN
    EXEC sp_executesql @sql;
END


