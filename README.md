
### 15th July 2024 C (Spercailty )

DECLARE @sql NVARCHAR(MAX) = N'';
DECLARE @table NVARCHAR(128);
DECLARE @schema NVARCHAR(128);
DECLARE @specialtyColumn NVARCHAR(128);

-- Cursor to iterate through all relevant tables and columns
DECLARE column_cursor CURSOR FOR
SELECT 
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c.COLUMN_NAME AS SpecialtyColumn
FROM 
    INFORMATION_SCHEMA.TABLES t
JOIN 
    INFORMATION_SCHEMA.COLUMNS c ON t.TABLE_SCHEMA = c.TABLE_SCHEMA AND t.TABLE_NAME = c.TABLE_NAME
WHERE 
    t.TABLE_TYPE = 'BASE TABLE'
    AND c.COLUMN_NAME = 'Specialty';

-- Open the cursor
OPEN column_cursor;
FETCH NEXT FROM column_cursor INTO @schema, @table, @specialtyColumn;

-- Loop through the cursor
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = @sql + 
        'SELECT ''' + @schema + '.' + @table + ''' AS TableName, ' +
        QUOTENAME(@specialtyColumn) + ' AS Specialty ' +
        'FROM ' + QUOTENAME(@schema) + '.' + QUOTENAME(@table) + ' ' +
        'UNION ALL ';
        
    FETCH NEXT FROM column_cursor INTO @schema, @table, @specialtyColumn;
END;

-- Close and deallocate the cursor
CLOSE column_cursor;
DEALLOCATE column_cursor;

-- Remove the last 'UNION ALL'
IF LEN(@sql) > 0
BEGIN
    SET @sql = LEFT(@sql, LEN(@sql) - LEN(' UNION ALL '));
END

-- Print the final dynamic SQL for debugging
PRINT @sql;

-- Execute the dynamic SQL
EXEC sp_executesql @sql;

===================================================================================================================================================================
### 15th July 2024 B
=========================================================================================================================================================================
DECLARE @sql NVARCHAR(MAX) = N'';
DECLARE @table NVARCHAR(128);
DECLARE @schema NVARCHAR(128);
DECLARE @column1 NVARCHAR(128);
DECLARE @column2 NVARCHAR(128);
DECLARE @specialtyColumn NVARCHAR(128);

-- Cursor to iterate through all relevant tables and columns
DECLARE column_cursor CURSOR FOR
SELECT 
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c1.COLUMN_NAME AS SurnameColumn,
    c2.COLUMN_NAME AS FirstnameColumn,
    s.COLUMN_NAME AS SpecialtyColumn
FROM 
    INFORMATION_SCHEMA.TABLES t
JOIN 
    INFORMATION_SCHEMA.COLUMNS c1 ON t.TABLE_SCHEMA = c1.TABLE_SCHEMA AND t.TABLE_NAME = c1.TABLE_NAME
JOIN 
    INFORMATION_SCHEMA.COLUMNS c2 ON t.TABLE_SCHEMA = c2.TABLE_SCHEMA AND t.TABLE_NAME = c2.TABLE_NAME
JOIN 
    INFORMATION_SCHEMA.COLUMNS s ON t.TABLE_SCHEMA = s.TABLE_SCHEMA AND t.TABLE_NAME = s.TABLE_NAME AND s.COLUMN_NAME = 'Specialty'
WHERE 
    t.TABLE_TYPE = 'BASE TABLE'
    AND (c1.COLUMN_NAME = 'Surname' OR c1.COLUMN_NAME = 'FamilyName')
    AND c2.COLUMN_NAME = 'Firstname';

-- Open the cursor
OPEN column_cursor;
FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2, @specialtyColumn;

-- Loop through the cursor
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = @sql + 
        'SELECT ''' + @schema + '.' + @table + ''' AS TableName, ' +
        'COALESCE(' + QUOTENAME(@column1) + ', ' + QUOTENAME(@column1) + ') AS LastName, ' +
        QUOTENAME(@column2) + ' AS Firstname, ' +
        'COALESCE(' + QUOTENAME(@specialtyColumn) + ', ''No Specialty'') AS User_Specialty ' +
        'FROM ' + QUOTENAME(@schema) + '.' + QUOTENAME(@table) + ' ' +
        'WHERE ' + QUOTENAME(@column1) + ' IS NOT NULL ' +
        'UNION ALL ';
        
    FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2, @specialtyColumn;
END;

-- Close and deallocate the cursor
CLOSE column_cursor;
DEALLOCATE column_cursor;

-- Remove the last 'UNION ALL'
IF LEN(@sql) > 0
BEGIN
    SET @sql = LEFT(@sql, LEN(@sql) - LEN(' UNION ALL '));
END

-- Print the final dynamic SQL for debugging
PRINT @sql;

-- Execute the dynamic SQL
EXEC sp_executesql @sql;

==============================================================================================================================================
### 15th July 2024 A
===============================================================================================================================================
DECLARE @sql NVARCHAR(MAX) = N'';
DECLARE @table NVARCHAR(128);
DECLARE @schema NVARCHAR(128);
DECLARE @column1 NVARCHAR(128);
DECLARE @column2 NVARCHAR(128);
DECLARE @specialtyColumn NVARCHAR(128);

-- Cursor to iterate through all relevant tables and columns
DECLARE column_cursor CURSOR FOR
SELECT 
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c1.COLUMN_NAME AS SurnameColumn,
    c2.COLUMN_NAME AS FirstnameColumn,
    c3.COLUMN_NAME AS SpecialtyColumn
FROM 
    INFORMATION_SCHEMA.TABLES t
JOIN 
    INFORMATION_SCHEMA.COLUMNS c1 ON t.TABLE_SCHEMA = c1.TABLE_SCHEMA AND t.TABLE_NAME = c1.TABLE_NAME
JOIN 
    INFORMATION_SCHEMA.COLUMNS c2 ON t.TABLE_SCHEMA = c2.TABLE_SCHEMA AND t.TABLE_NAME = c2.TABLE_NAME
LEFT JOIN 
    INFORMATION_SCHEMA.COLUMNS c3 ON t.TABLE_SCHEMA = c3.TABLE_SCHEMA AND t.TABLE_NAME = c3.TABLE_NAME AND c3.COLUMN_NAME = 'Specialty'
WHERE 
    t.TABLE_TYPE = 'BASE TABLE'
    AND (c1.COLUMN_NAME = 'Surname' OR c1.COLUMN_NAME = 'FamilyName')
    AND c2.COLUMN_NAME = 'Firstname';

-- Open the cursor
OPEN column_cursor;
FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2, @specialtyColumn;

-- Loop through the cursor
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = @sql + 
        'SELECT ''' + @schema + '.' + @table + ''' AS TableName, ' +
        'COALESCE(' + QUOTENAME(@column1) + ', ' + QUOTENAME(@column1) + ') AS LastName, ' +
        QUOTENAME(@column2) + ' AS Firstname, ' +
        'CASE WHEN ' + QUOTENAME(@specialtyColumn) + ' IS NOT NULL THEN ' + QUOTENAME(@specialtyColumn) + ' ELSE NULL END AS User_Specialty ' +
        'FROM ' + QUOTENAME(@schema) + '.' + QUOTENAME(@table) + ' ' +
        'WHERE ' + QUOTENAME(@column1) + ' IS NOT NULL ' +
        'UNION ALL ';
        
    FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2, @specialtyColumn;
END;

-- Close and deallocate the cursor
CLOSE column_cursor;
DEALLOCATE column_cursor;

-- Remove the last 'UNION ALL'
IF LEN(@sql) > 0
BEGIN
    SET @sql = LEFT(@sql, LEN(@sql) - LEN(' UNION ALL '));
END

-- Execute the dynamic SQL
EXEC sp_executesql @sql;

========================================================================================================================================================================================================================

It is possible to create a dynamic SQL query that iterates through all the tables in your database to extract the `Surname` or `FamilyName` and `Firstname` columns, regardless of their naming inconsistencies. This can be achieved by querying the system catalog views to identify the relevant columns and then constructing and executing a dynamic SQL statement.

Here's how you can do it:

1. **Identify the relevant columns across all tables using system catalog views.**
2. **Construct a dynamic SQL query that iterates through these tables and columns.**
3. **Execute the dynamic SQL to get the desired result.**

Here's the T-SQL code to accomplish this:

```sql
DECLARE @sql NVARCHAR(MAX) = N'';
DECLARE @table NVARCHAR(128);
DECLARE @schema NVARCHAR(128);
DECLARE @column1 NVARCHAR(128);
DECLARE @column2 NVARCHAR(128);

-- Cursor to iterate through all relevant tables and columns
DECLARE column_cursor CURSOR FOR
SELECT 
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c1.COLUMN_NAME AS SurnameColumn,
    c2.COLUMN_NAME AS FirstnameColumn
FROM 
    INFORMATION_SCHEMA.TABLES t
JOIN 
    INFORMATION_SCHEMA.COLUMNS c1 ON t.TABLE_SCHEMA = c1.TABLE_SCHEMA AND t.TABLE_NAME = c1.TABLE_NAME
JOIN 
    INFORMATION_SCHEMA.COLUMNS c2 ON t.TABLE_SCHEMA = c2.TABLE_SCHEMA AND t.TABLE_NAME = c2.TABLE_NAME
WHERE 
    t.TABLE_TYPE = 'BASE TABLE'
    AND (c1.COLUMN_NAME = 'Surname' OR c1.COLUMN_NAME = 'FamilyName')
    AND c2.COLUMN_NAME = 'Firstname';

-- Open the cursor
OPEN column_cursor;
FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2;

-- Loop through the cursor
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = @sql + 
        'SELECT ''' + @schema + '.' + @table + ''' AS TableName, ' +
        'COALESCE(' + QUOTENAME(@column1) + ', ' + QUOTENAME(@column1) + ') AS LastName, ' +
        QUOTENAME(@column2) + ' AS Firstname ' +
        'FROM ' + QUOTENAME(@schema) + '.' + QUOTENAME(@table) + ' ' +
        'WHERE ' + QUOTENAME(@column1) + ' IS NOT NULL ' +
        'UNION ALL ';
        
    FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2;
END;

-- Close and deallocate the cursor
CLOSE column_cursor;
DEALLOCATE column_cursor;

-- Remove the last 'UNION ALL'
IF LEN(@sql) > 0
BEGIN
    SET @sql = LEFT(@sql, LEN(@sql) - LEN(' UNION ALL '));
END

-- Execute the dynamic SQL
EXEC sp_executesql @sql;
```

### Explanation:
1. **Cursor Initialization**: A cursor is initialized to iterate through all relevant tables and columns.
2. **Joining System Catalog Views**: `INFORMATION_SCHEMA.TABLES` and `INFORMATION_SCHEMA.COLUMNS` views are used to identify tables that contain either `Surname` or `FamilyName` and `Firstname` columns.
3. **Dynamic SQL Construction**: For each table identified, a `SELECT` statement is constructed dynamically, appending it to the `@sql` variable.
4. **Removing Last `UNION ALL`**: The trailing `UNION ALL` is removed to ensure the final query is valid.
5. **Executing the Query**: The constructed dynamic SQL is executed using `sp_executesql`.

### This method dynamically builds and executes the SQL query to handle a large number of tables without manually specifying each table, making it efficient and scalable for databases with many tables.
=====================================================================================================================
To modify the given query so that it inserts the output into a table called `mion.UnusedUsers` in a database called `InfoT`, you can wrap the dynamically constructed query in an `INSERT INTO` statement. Here’s how you can do it:

1. **Create the target table if it doesn't exist**.
2. **Modify the dynamic SQL to insert the results into the target table**.

Here is the updated script:

```sql
USE InfoTeamArea;

-- Step 1: Create the target table if it doesn't exist
IF OBJECT_ID('mion.UnusedUsers', 'U') IS NULL
BEGIN
    CREATE TABLE mion.UnusedUsers (
        TableName NVARCHAR(256),
        LastName NVARCHAR(256),
        Firstname NVARCHAR(256)
    );
END;

-- Step 2: Declare variables for dynamic SQL construction
DECLARE @sql NVARCHAR(MAX) = N'';
DECLARE @table NVARCHAR(128);
DECLARE @schema NVARCHAR(128);
DECLARE @column1 NVARCHAR(128);
DECLARE @column2 NVARCHAR(128);

-- Step 3: Cursor to iterate through all relevant tables and columns
DECLARE column_cursor CURSOR FOR
SELECT 
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c1.COLUMN_NAME AS SurnameColumn,
    c2.COLUMN_NAME AS FirstnameColumn
FROM 
    INFORMATION_SCHEMA.TABLES t
JOIN 
    INFORMATION_SCHEMA.COLUMNS c1 ON t.TABLE_SCHEMA = c1.TABLE_SCHEMA AND t.TABLE_NAME = c1.TABLE_NAME
JOIN 
    INFORMATION_SCHEMA.COLUMNS c2 ON t.TABLE_SCHEMA = c2.TABLE_SCHEMA AND t.TABLE_NAME = c2.TABLE_NAME
WHERE 
    t.TABLE_TYPE = 'BASE TABLE'
    AND (c1.COLUMN_NAME = 'Surname' OR c1.COLUMN_NAME = 'FamilyName')
    AND c2.COLUMN_NAME = 'Firstname';

-- Step 4: Open the cursor
OPEN column_cursor;
FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2;

-- Step 5: Loop through the cursor to build the dynamic SQL
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = @sql + 
        'SELECT ''' + @schema + '.' + @table + ''' AS TableName, ' +
        'COALESCE(' + QUOTENAME(@column1) + ', ' + QUOTENAME(@column1) + ') AS LastName, ' +
        QUOTENAME(@column2) + ' AS Firstname ' +
        'FROM ' + QUOTENAME(@schema) + '.' + QUOTENAME(@table) + ' ' +
        'WHERE ' + QUOTENAME(@column1) + ' IS NOT NULL ' +
        'UNION ALL ';
        
    FETCH NEXT FROM column_cursor INTO @schema, @table, @column1, @column2;
END;

-- Step 6: Close and deallocate the cursor
CLOSE column_cursor;
DEALLOCATE column_cursor;

-- Step 7: Remove the last 'UNION ALL'
IF LEN(@sql) > 0
BEGIN
    SET @sql = LEFT(@sql, LEN(@sql) - LEN(' UNION ALL '));
END

-- Step 8: Insert the results into the target table
SET @sql = 'INSERT INTO mion.UnusedUsers (TableName, LastName, Firstname) ' + @sql;

-- Step 9: Execute the dynamic SQL
EXEC sp_executesql @sql;
```

### Explanation:

1. **Creating the Target Table**: The script first checks if the `mion.UnusedUsers` table exists in the `InfoTeamArea` database. If it doesn’t, it creates the table.
2. **Dynamic SQL Construction**: The cursor iterates through all relevant tables and columns, constructing a `SELECT` statement for each table that matches the criteria.
3. **Combining Queries with `UNION ALL`**: The individual `SELECT` statements are combined using `UNION ALL`.
4. **Removing the Last `UNION ALL`**: The trailing `UNION ALL` is removed to ensure the final query is valid.
5. **Inserting Results into Target Table**: The dynamically constructed query is prefixed with an `INSERT INTO` statement to insert the results into `mion.UnusedUsers`.
6. **Executing the Query**: The final dynamic SQL query is executed.

###This approach ensures that the results from all relevant tables are inserted into the specified table in a single operation.
=======================================================================================================================
### Optimized Algorithm with Unique Identifier:

1. **Create a Permanent Table to Store Extracted User Data:**
    - Create a permanent table `MPI_ExtractedUsers` in the `MPI_DataWarehouse` to store user information extracted from relevant tables.
    - Ensure this table includes a unique identifier column (e.g., `UserID`).

2. **Identify Relevant Tables and Columns:**
    - Dynamically identify all relevant columns across the 5000 tables that might contain user-related information such as `Username`, `FamilyName`, `FirstName`, etc.
    - Include a step to extract the primary key or unique identifier for each user record to maintain data integrity.

3. **Extract User Data:**
    - Extract user data from identified columns and populate the `MPI_ExtractedUsers` table.
    - Ensure that each user record includes the unique identifier (`UserID`).

4. **Update `LastLogin` and `Specialty` Fields:**
    - Update the `LastLogin` and `Specialty` fields by checking all columns in the tables that contain login information.
    - Use the unique identifier to ensure that updates are applied to the correct user records.

5. **Update `LastSeen` Field:**
    - For users with `LastLogin` missing (NULL), update the `LastSeen` field by checking the audit trail or relevant tables.
    - Again, use the unique identifier to match and update records accurately.

6. **Update `User_Type` Field:**
    - Populate the `User_Type` field, setting it to "External" for users where `User_Type` remains NULL.
    - Use the unique identifier to ensure accurate updates.

7. **Update `Band` Field:**
    - Derive the `Band` column based on `LastLogin` or `LastSeen`. If the last activity is less than 12 months, set the `Band` as "Assumed Current"; otherwise, set it as "Assumed to be unused".
    - Utilize the unique identifier to ensure the correct records are updated.

### Optimization Considerations:

- **Indexing:** Ensure that the `UserID` column in the `MPI_ExtractedUsers` table is indexed to improve the performance of update operations.
- **Batch Processing:** Process the data in batches to manage memory usage and improve performance, especially given the large number of tables and records.
- **Parallel Processing:** If possible, use parallel processing to handle different sections of the data simultaneously, reducing overall processing time.
- **Error Handling:** Implement robust error handling and logging to track any issues that arise during the extraction and update processes.
- **Data Validation:** Include steps to validate the data before and after updates to ensure data integrity and consistency


### Flow chart Mermaid code:
graph TD
    A[Start] --> B[Create Users Table with ID]
    B --> C[Identify Relevant Tables and Columns]
    C --> D[Extract User Data]
    D --> E[Store Data in Users Table]
    E --> F[Update LastLogin and Specialty]
    F --> G[Ensure Accurate Updates]
    G --> H[Update LastSeen for Missing LastLogin]
    H --> I[Ensure Accurate Updates]
    I --> J[Set User_Type to External where NULL]
    J --> K[Ensure Accurate Updates]
    K --> L[Derive and Update Band Field]
    L --> M[Ensure Accurate Updates]
    M --> N[End]


### Instructions to Use:
Open draw.io: Go to draw.io and start a new diagram.
Create New Diagram: Choose a blank diagram or any template that suits your needs.
Insert Mermaid Code:
Go to Insert > Advanced > Mermaid.
Paste the above code into the dialog box.
Render the Diagram: Click Insert and the diagram will be rendered based on the Mermaid code.
