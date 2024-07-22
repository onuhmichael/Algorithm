### UnusedUsers Algorithm in a Clinical System 22/07/24.

1. **Tables Involved**:
   - `GenPerson`
   - `GenUser`
   - `GenUserAccessHistory`

2. **Columns to Select**:
   - From `GenPerson`: `GenUserID`, `Surname`, `FirstName`
   - From `GenUserAccessHistory`: `LogDateTime`
   - Add a new column: `CurrentDateTime` (which will be the current date and time)

3. **Join Conditions**:
   - Join `GenPerson` and `GenUser` on their common column.
   - Join `GenUser` and `GenUserAccessHistory` on `UserNumber`.

4. **Target Table**:
   - A new table where the selected columns and the new `CurrentDateTime` column will be inserted.

Let's name the new table `GenUserAccessLog`.

Here is the T-SQL code that performs the required operations:

```sql
-- Step 1: Create the new table GenUserAccessLog
CREATE TABLE GenUserAccessLog (
    GenUserID INT,
    Surname NVARCHAR(100),
    FirstName NVARCHAR(100),
    LogDateTime DATETIME,
    CurrentDateTime DATETIME
);

-- Step 2: Insert data into the new table
INSERT INTO GenUserAccessLog (GenUserID, Surname, FirstName, LogDateTime, CurrentDateTime)
SELECT
    gp.GenUserID,
    gp.Surname,
    gp.FirstName,
    guah.LogDateTime,
    GETDATE() AS CurrentDateTime
FROM
    GenPerson gp
JOIN
    GenUser gu ON gp.GenUserID = gu.GenUserID
JOIN
    GenUserAccessHistory guah ON gu.UserNumber = guah.UserNumber;
```

### Explanation:

1. **Creating the New Table**:
   - The `CREATE TABLE` statement defines the structure of the new table `GenUserAccessLog`.

2. **Inserting Data**:
   - The `INSERT INTO` statement is used to populate the `GenUserAccessLog` table.
   - The `SELECT` statement gathers the required columns from the `GenPerson`, `GenUser`, and `GenUserAccessHistory` tables using JOIN operations.
   - `GETDATE()` function is used to insert the current date and time into the `CurrentDateTime` column.

If you need any modifications or have further questions, please let me know!
