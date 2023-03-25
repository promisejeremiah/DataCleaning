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
