**# Student Performance Assignment - Part 1 Report**



**Hello Sir, this is my report for Part 1 (Data Acquisition, Cleaning, and EDA). I have successfully completed all 10 tasks in the notebook and cleaned the dataset. Below are the key findings from my analysis.**



**## 1. Missing Values \& Imputation Strategy**

**\* For the \*\*Exam\_Score\*\* column, the mean is 67.23 and the median is 67.00. Since they are almost the same, this column has a balanced and normal distribution.**

**\* For the \*\*Tutoring\_Sessions\*\* column, the mean is 1.49 but the median is 1.00. The mean is slightly higher because some students took a lot of sessions (positive skewness).**

**\* \*\*My Decision:\*\* Because of this skewness, I chose the \*\*Median\*\* to fill the missing values using the `.fillna()` method. A final check confirms there are now 0 null values remaining in the data.**



**## 2. Duplicate Data Cleaning**

**\* I checked for duplicate rows using `df.duplicated().sum()`.** 

**\* All identical duplicate rows were removed using `df.drop\_duplicates()` to make sure the data is clean and unique before making any plots.**



**## 3. Correlation Heatmap Observations**

**\* The strongest relationship found is between \*\*Attendance\*\* and \*\*Exam\_Score\*\* (correlation value is \*\*0.58\*\*). Another important relationship is between \*\*Hours\_Studied\*\* and \*\*Exam\_Score\*\* (\*\*0.45\*\*).**

**\* \*\*Alternative Explanation:\*\* While good attendance helps in getting better marks, an alternative reason could be "Student Motivation." Highly motivated students will naturally go to college daily (high attendance) and also study hard at home, which leads to higher exam scores.**



**## 4. Spearman vs Pearson Correlation**

**\* When comparing both methods, the top 3 differences were found in these pairs:**

&#x20; **1. Attendance \& Exam\_Score (Spearman: 0.6724, Pearson: 0.5800 -> Difference: 0.0924)**

&#x20; **2. Hours\_Studied \& Exam\_Score (Spearman: 0.4810, Pearson: 0.4500 -> Difference: 0.0310)**

&#x20; **3. Previous\_Scores \& Exam\_Score (Spearman: 0.1919, Pearson: 0.1800 -> Difference: 0.0119)**

**\* \*\*Conclusion for Part 2:\*\* The big difference in Attendance shows that the relationship is non-linear but moving in one direction. Therefore, we should use \*\*Spearman Rank Correlation\*\* for selecting features in Part 2.**



**## 5. Grouped Aggregation Insights**

**\* Students with \*\*"High"\*\* Parental Involvement have the highest average score (\*\*68.09\*\*).**

**\* Students with \*\*"Low"\*\* Parental Involvement have the highest standard deviation (\*\*3.97\*\*), meaning their marks vary the most.**

**\* The ratio between the highest and lowest group mean is \*\*1.0261\*\* ($68.0928 / 66.3583$). Even though it is close to 1, the clear difference shows that Parental Involvement provides a good predictive signal for the model.**



**## 6. Final Saved Data**

**\* The cleaned data has been successfully saved as \*\*`cleaned\_data.csv`\*\* without index columns, and it is ready to be used in Part 2 and Part 3.**

**\***

