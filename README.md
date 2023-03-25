# DataCleaning


So i joined the data cleaning exercise, to clean up this messy dataset.

I used SQL as my preferred tool for the data cleaning exercise.

Here i will go through the steps i used to clean this dataset.

After downloading and installing Microsoft SQL server to my laptop, i uploaded the dataset into the SQL server.

The first thing i did was to take a look at the dataset by running the following command.

```sql
SELECT * FROM dbo.fifa;
```
![Screenshot 2023-03-25 at 1 42 19 PM](https://user-images.githubusercontent.com/48945500/227718052-c8fd0403-0a36-466f-83fa-5f17e8132bd8.png)

After running the command we can see that our dataset contains 77 columns and 18980 rows.
- Some of the club in the Club column begins with an integer value of 1.
- The Joined column is in the DATETIME — format and needs be converted to the DATE-format.
- The Contract column needs to be splited into two different columns and populated with needed values.
- In the Positions column some of the players has more than one position.
- In the Height column some records contains values in feet-inches and others contain values in kg. All the records in the Height column needs to be consistent.
- The Weight column contains records that are in lbs and others in CM It needs to be consistent.
- The Value, Wage and Release_Clause columns contains the € sign and values that are in millions and represented as M while values that are in thousand are represented as K. It will be stripped off those special characters and where theres an M it will be multiplied by 1,000,000 and K 1,000.
- The W_F, SM and IR columns all contains the ★ icon which signifies the star rating of each player in that column. The ★ icon will be stripped and all columns converted to INT.
- Some records in the Hits columns contains NULL values and a K character which means such record is in thousands. The NULLS will be replaced with a 0 and where there’s a K, it will be stripped and multiplied by 1000.
- Some columns have the wrong data type and needs to be changed to the correct data type.
- Some columns will need to be renamed.
- Some columns will be dropped since they have no meaning to the dataset.

&nbsp;
&nbsp;
&nbsp;
&nbsp;

1. CLUB COLUMN
I will start with cleaning the Club column.

```sql
SELECT Club
FROM dbo.fifa
WHERE Club LIKE '%1.%';
```
After running the folowing query we can see that the Club column we can see that some club contains some unwanted character.

![image](https://user-images.githubusercontent.com/48945500/227719160-9164ffc0-ee5f-4917-88f1-bd2054f55f97.png)

Now we are going to get rid of the unwanted character in the Club column with the following code.

```sql
UPDATE fifa
SET Club = CASE WHEN Club LIKE '%1.%' THEN REPLACE(REPLACE(Club, '.', ''), '1', '') 
ELSE Club 
END;
```

&nbsp;
&nbsp;
&nbsp;
&nbsp;

2. JOINED COLUMN

```sql
SELECT Joined FROM dbo.fifa;
```
![Screenshot 2023-03-25 at 2 13 52 PM](https://user-images.githubusercontent.com/48945500/227719364-2653bab4-04ef-4c43-bc9f-060bfa43b0da.png)


Convert the Joined column from DATETIME — format to DATE — format and create a new column for it.

```sql
ALTER TABLE fifa
ADD Joined_Date DATE;
UPDATE fifa
SET Joined_Date = CONVERT(DATE, Joined);
```

![image](https://user-images.githubusercontent.com/48945500/227719409-34d3b203-2cb1-4e37-a20f-dea3a915a7cd.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

3. CONTRACT COLUMN

```sql
SELECT Contract FROM dbo.fifa;
```
![Screenshot 2023-03-25 at 2 18 48 PM](https://user-images.githubusercontent.com/48945500/227719534-cc308020-cc88-4b78-921a-49bb878f9c24.png)

Split the Contract column to Contract_Start and Contract_End, create two new columns for it, replace unwanted character and drop rows that are not needed.

```sql
--Create two new columns
ALTER TABLE fifa
ADD Contract_Start VARCHAR(50), Contract_End VARCHAR(50);

--Split the contract column into the newly created columns
UPDATE fifa
 SET Contract_Start = 
  SUBSTRING(Contract, 1, ABS(CHARINDEX('~', Contract) -1));
UPDATE fifa
 SET Contract_End =
  SUBSTRING(Contract, CHARINDEX('~', Contract) +1, LEN(Contract));
  ```
  
  ![image](https://user-images.githubusercontent.com/48945500/227719571-5c237f84-aed4-4db3-a38e-246af5ce1ed5.png)

After splitting the Contract columns some rows some rows in the contract_start column contains unwanted values. 

```sql
SELECT Contract_Start FROM dbo.fifa WHERE LEN(Contract_Start) = 1;
```

![image](https://user-images.githubusercontent.com/48945500/227719710-626dfbe1-fba7-48a7-8805-c187da710034.png)

We  fill it up the with the YEAR from Joined_date column since the contract of any player starts the year he joined the team.

```sql
UPDATE fifa
SET Contract_Start = YEAR(Joined_Date)
WHERE LEN(Contract_Start) = 1;
```

Some rows in the Contract_End column contains values the value ‘On Loan’ and the date value is same with the Loan_date_end column. 

```sql
SELECT Loan_Date_End, Contract_End FROM dbo.fifa WHERE Contract_End LIKE '%On Loan';
```
![image](https://user-images.githubusercontent.com/48945500/227719828-b1d05a65-3078-4e01-a3a5-73bfabc0f5b6.png)

We replace the ‘On Loan’ value in the Contract_End column with the YEAR from the Loan_date_end column.
```sql
UPDATE fifa
SET Contract_End = YEAR(Loan_Date_End)
WHERE Contract_End LIKE '%On Loan';
```
Some rows in the Contract_End column contains the value ‘Free’ and we will not be needed it in our dataset because rows that contains such value have 0.00 values in the Value, Wage, Release_Clause columns and also contains ‘No club’ in the Club column.

```sql
SELECT Contract_End, Club, Value, Wage, Release_Clause FROM dbo.fifa WHERE Contract_End = 'Free';
```
![image](https://user-images.githubusercontent.com/48945500/227719984-007d2732-73d2-4b61-9e00-b1f892310629.png)

we will delete such rows.
```sql
DELETE FROM dbo.fifa WHERE Contract_End = 'Free';
```
&nbsp;
&nbsp;
&nbsp;
&nbsp;

4. POSITIONS COLUMN

```sql
SELECT Positions FROM dbo.fifa;
```
The Positions column contains different positions of the players and we will be needing just one position alone. So we get rid of other positions and keep the first position alone.

![Screenshot 2023-03-25 at 2 30 51 PM](https://user-images.githubusercontent.com/48945500/227720201-d6f3d9d2-ca41-4b63-b1ff-f9c77b5a6e9c.png)

```sql
UPDATE fifa
SET Positions = CASE 
WHEN Positions LIKE '%,%' THEN SUBSTRING(Positions, 1, ABS(CHARINDEX(',', Positions) -1))
ELSE Positions 
END;
```
![image](https://user-images.githubusercontent.com/48945500/227720245-084a5c49-af1b-4ed9-9d05-807e44536f38.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

5. HEIGHT COLUMN

```sql
SELECT DISTINCT Height, COUNT(Height) FROM dbo.fifa GROUP BY Height;
```
![Screenshot 2023-03-25 at 2 42 27 PM](https://user-images.githubusercontent.com/48945500/227720872-7347b90a-cd66-4cd0-b203-6b1d70747613.png)

Some rows in the Height columns are measured in feet-inches so we convert the it from feet-inches to CM.
```sql
UPDATE fifa
SET Height = CASE 
        WHEN Height LIKE '%''%"' THEN 
        TRY_CONVERT(DECIMAL(10,0), SUBSTRING(Height, 1, CHARINDEX('''', Height) -1))*30.48 + 
        TRY_CONVERT(DECIMAL(10,0), SUBSTRING(Height, 1, LEN(Height) -2))*2.54
        ELSE TRY_CONVERT(DECIMAL(10,0), SUBSTRING(Height, 1, LEN(Height) -2)) 
    END;
```
![image](https://user-images.githubusercontent.com/48945500/227721116-dac08466-2b9c-4964-8544-c4f84a9e2a52.png)


&nbsp;
&nbsp;
&nbsp;
&nbsp;

6. WEIGHT COLUMN

```sql
SELECT DISTINCT Weight, COUNT(Weight) AS Weight_count FROM dbo.fifa GROUP BY Weight;
```
![Screenshot 2023-03-25 at 2 45 16 PM](https://user-images.githubusercontent.com/48945500/227721021-025ed6a1-8835-4cdb-9569-efeb83c2bb51.png)

Some rows in the Weight columns are measured in 'lbs so we convert the it from 'lbs to KG.

```sql
UPDATE fifa
SET Weight = CASE WHEN Weight LIKE '%lbs' THEN TRY_CONVERT (DECIMAL(10,0), SUBSTRING(Weight, 1, LEN(Weight) -3)) * 0.45
        ELSE TRY_CONVERT (DECIMAL(10,0), SUBSTRING(Weight, 1, LEN(Weight) -2))
 END;
```
![image](https://user-images.githubusercontent.com/48945500/227721162-f7f6fd9a-3326-49ae-8fda-61f79dfe97bb.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;


7. VALUE COLUMN
```sql
SELECT DISTINCT Value, COUNT(Value) AS Value_count FROM dbo.fifa GROUP BY Value;
```
![Screenshot 2023-03-25 at 3 04 09 PM](https://user-images.githubusercontent.com/48945500/227721994-b2383428-c889-4994-ae37-435b6cb65050.png)

Some of the rows in the Value column contains the € sign and values that are in millions and represented as M while values that are in thousand are represented as K. It will be stripped off those special characters and where theres an M it will be multiplied by 1,000,000 and K 1,000.

```sql
UPDATE fifa
 SET Value = CASE
  WHEN Value LIKE '€%' AND Value LIKE '%M' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Value, '€', ''), 'M', '')) * 1000000
  WHEN Value LIKE '€%' AND Value LIKE '%K' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Value, '€', ''), 'K', '')) * 1000
  WHEN Value LIKE '€%' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(Value, '€', ''))
  ELSE Value
 END;
```
![image](https://user-images.githubusercontent.com/48945500/227721411-bc2e100b-797b-47b9-8918-0e8a9c968fd9.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

8. WAGE COLUMN

```sql
SELECT DISTINCT Wage, COUNT(Wage) AS Wage_count FROM dbo.fifa GROUP BY Wage;
```
![Screenshot 2023-03-25 at 3 02 49 PM](https://user-images.githubusercontent.com/48945500/227721909-8f3c819d-9402-41b6-9504-74cec420c616.png)

Some of the Value column contains the € sign and values that are in thousand are represented as K. It will be stripped off those special characters and where theres an M it will be multiplied by 1,000,000 and K 1,000.

```sql
UPDATE fifa
SET Wage = CASE
    WHEN Wage LIKE '€%' AND Wage LIKE '%K' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Wage, '€', ''), 'K', '')) * 1000
    WHEN Wage LIKE '€%' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(Wage, '€', ''))
    ELSE Wage
END;
```

&nbsp;
&nbsp;
&nbsp;
&nbsp;

9. RELEASE_CLAUSE COLUMN

```sql
SELECT DISTINCT Release_Clause, COUNT(Release_Clause) AS Release_Clause_count FROM dbo.fifa GROUP BY Release_Clause;
```
![Screenshot 2023-03-25 at 3 05 34 PM](https://user-images.githubusercontent.com/48945500/227722066-1afb8d55-759f-473c-b180-b9b36b91a66f.png)

Some of the rows in the Release_Clause column contains the € sign and values that are in millions and represented as M while values that are in thousand are represented as K. It will be stripped off those special characters and where theres an M it will be multiplied by 1,000,000 and K 1,000.

```sql
UPDATE fifa
 SET Release_Clause = CASE
  WHEN Release_Clause LIKE '€%' AND Release_Clause LIKE '%M' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Release_Clause, '€', ''), 'M', '')) * 1000000
  WHEN Release_Clause LIKE '€%' AND Release_Clause LIKE '%K' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Release_Clause, '€', ''), 'K', '')) * 1000
  WHEN Release_Clause LIKE '€%' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(Release_Clause, '€', ''))
  ELSE Release_Clause
 END;
```
![image](https://user-images.githubusercontent.com/48945500/227722165-a84ed3b7-aa81-4d50-9f4f-475a571ba3ff.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

10. W_F COLUMN

```sql
SELECT W_F FROM dbo.fifa;
```
![Screenshot 2023-03-25 at 3 16 21 PM](https://user-images.githubusercontent.com/48945500/227722696-a4b00e56-3b14-41e6-9912-2615b2fbc154.png)

The W_F column contains some special characters ★ and we will get rid of it.
```sql
UPDATE fifa
SET W_F = SUBSTRING(W_F, 1, CHARINDEX(' ', W_F) -1);
```
![image](https://user-images.githubusercontent.com/48945500/227722756-d099db26-9c92-4bf5-b3d5-4b1b05757325.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

11. SM COLUMN

```sql
SELECT W_F FROM dbo.fifa;
```
![Screenshot 2023-03-25 at 3 18 53 PM](https://user-images.githubusercontent.com/48945500/227722842-b143d8ed-5ea1-47ce-adb9-5f620391f5dd.png)

The SM column contains some special characters ★ and we will get rid of it.

```sql
UPDATE fifa
SET SM = SUBSTRING(SM, 1, CHARINDEX(' ', SM) +1);
```
![image](https://user-images.githubusercontent.com/48945500/227722903-cc20969e-0579-4f76-a25b-ff5a3d7f07b7.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

12. IR COLUMN

```sql
SELECT IR FROM dbo.fifa;
```
![Screenshot 2023-03-25 at 3 21 35 PM](https://user-images.githubusercontent.com/48945500/227722987-c8f2660e-2568-4951-a294-6405ead4000b.png)

The IR column contains some special characters ★ and we will get rid of it.

```sql
UPDATE fifa
SET IR = SUBSTRING(IR, 1, CHARINDEX(' ', IR) -1);
```
![image](https://user-images.githubusercontent.com/48945500/227723026-c0b077b6-fd93-49ce-8aaa-4db98af173f2.png)

&nbsp;
&nbsp;
&nbsp;
&nbsp;

13. HITS COLUMN

```sql
SELECT DISTINCT Hits, COUNT(Hits) AS Hits_count FROM dbo.fifa GROUP BY Hits ORDER BY Hits;
```
![Screenshot 2023-03-25 at 3 25 28 PM](https://user-images.githubusercontent.com/48945500/227723266-fab44d91-c6d8-4b43-9eb0-7c76e6075703.png)

The Hits column contains a ‘k’ character and some nulls. We will strip the ‘k’ and multiply by 1000 and also fill the nulls with 0.

```sql
UPDATE fifa
SET Hits = CASE
    WHEN Hits LIKE '%k' THEN TRY_CONVERT(DECIMAL(10,0), SUBSTRING(Hits,1, LEN(Hits) -1)) *1000
    WHEN Hits IS NULL THEN 0
    ELSE TRY_CONVERT(DECIMAL(10,0), Hits)
 END;
```
![image](https://user-images.githubusercontent.com/48945500/227723308-433dda89-deb1-4e72-827e-06ca63c6e80e.png)


