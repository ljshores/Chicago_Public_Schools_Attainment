# Understanding Academic Attainment in Chicago Public Schools

Ever wondered what makes some schools shine brighter than others regarding academic performance? In this project, this is exactly the question that I dive into. Using data from Chicago Public Schools (CPS) and neighborhood demographic data, I attempt to uncover insights into the ingredients of academic success—and the obstacles standing in the way.


## Problem Statement: The Big Questions
What factors influence academic attainment in schools?
Does a school’s neighborhood factor into its performance?

## Approach

In this project, I analyzed the school progress report data to find insights and modeled academic attainment using regression. The ultimate point here was to fit the best model possible in order to understand the key variables making up the variation in attainment.

The dataset included Chicago elementary, middle, and high schools. All variables in the data were not relevant to all school types. There were attainment scores for grades 2-8, as well as SAT and PSAT related scores for high school.

I decided to focus on elementary schools, since there are more of them and they would provide more data points. I decided to make 8th grade reading attainment scores the target variable (our measure of success). This made sense to me because it could proxy as a measure of how prepared the students were leaving the school, and because reading is often the basis of learning.

## Data Description

From CPS Progress Reports: 
Metrics on reading and math attainment, growth, attendance, student behavior, school culture, survey engagement, health, and creativity. 
654 schools total for the school year ending in 2018.
470 elementary schools. 418 schools after cleaning data for modeling purposes

2021 American Community Survey: 5-Year Data [2017-2021, Block Groups & Larger Areas]: 
Census block group-level details like median household income and racial composition.

Each school had lat/long information. So this was used to do a spatial merge with the corresponding census block group polygon (data was joined by geolocation).

**Things to be aware of:**
* Relatively small dataset of only 418 rows
* Autocorrelated features in the dataset, possible leakage if not aware what features using
* Census info tied to block groups might not reflect the school’s actual demographic makeup (not sure if the policy is that kids attend the school closest to them)

**Cleaning**
* Imputed some missing values for predictor variables
* Schools that did not contain the target variable (8th grade reading attainment) were deleted

**Feature Engineering**
* Created categorical representation of race variables. Binned into low, medium, and high according to cut-offs by a third (so, a census block group with less than .33 white population would be binned as low, etc.)
* Created variables that reflected the difference between a school’s metric score and the baseline (average) of that score


## Exploratory Data Analysis
<img width="570" alt="cps_map_es" src="https://github.com/user-attachments/assets/7b5551bd-644b-411b-9199-a5d798843c5f" />


### Methodology
Scatterplot Matrix and Correlation Matrix Heatmap to see how features relate to each other, and particularly our target
Histograms and boxplots to see the distributions of our numerical variables.
Boxplots to see the distributions of our target variable across the subgroups of other, categorical variables

### Key Points/Observations

* 8th grade reading attainment is higher on average than the lower grades and has a smaller range of variability.
  * Is this possibly because student performance converges as they learn more through school?
  * We don’t have info for class size, but it could be an effect of smaller classes/troubled students leaving school (by 8th grade) and thus leaving the higher performing student, thus raising the average
* Correlation between 2nd grade and 8th grade reading attainment is only .65, whereas correlation between 8th grade math and reading attainment is .85. Is this same-kids effect, or support for the point made above
* Neighborhood schools make up 74% of school, and have a super wide range in reading attainment
* Charter schools do NOT perform better than neighborhood schools in terms of top readers and avg readers
* Charter schools perform very similar to small schools, except that they have a lower floor for reading attainment



<img width="544" alt="school_type_ _attainment" src="https://github.com/user-attachments/assets/47eb8b05-2e89-4f7f-83d5-70d6ba13a901" />

<img width="276" alt="school_type_breakdown" src="https://github.com/user-attachments/assets/ed8eaf83-721f-4b5b-bba6-7a5af43dadbe" />


* Oddly enough, schools with very weak parent/teacher partnership had higher reading attainment levels
* Oddly enough, schools with very weak quality of facilities had higher reading attainment levels


<p float="left">
<img width="425" alt="Quality of Facilities" src="https://github.com/user-attachments/assets/0292c975-2fa4-456d-a47f-e05bbbf8b8ae" /> <img width="425" alt="Parent_Teacher_Partnershp" src="https://github.com/user-attachments/assets/35df0789-598f-422e-a95a-080eb2e81bfa" />
</p>


* There doesn't seem to be any strong correlations to attainment and race/income. Note that these census metrics are just for the block group that the school is located in...it's possible that this is too micro to be a good reflection of school makeup. Or even if people who live in that block group go to that school.
* There does seem to be some neighborhood effects on school attainment

<img width="1118" alt="neighborhood_boxplot_attianment" src="https://github.com/user-attachments/assets/422b3842-6836-4f99-8a2b-1a0a323429bc" />


* Although race variables did not show strong correlation to reading attainment, we do see some clear trends with attainment based on race. When we break race population makeup into bins, there are some clear indications that schools in higher % white block groups perform better with reading, and in higher % black block groups perform worse with reading

<p float="left">
<img width="400" alt="black_bins" src="https://github.com/user-attachments/assets/b5a9a382-8ee8-47ce-9ce5-6d6958ff0744" /> <img width="400" alt="white_bins" src="https://github.com/user-attachments/assets/d7bd4d99-5f79-4839-9c63-965cbad544e7" />
</p>



## Modeling

The main objective here was to understand the relationship between the features and the target. Could they accurately predict/explain what was going on in the target variable. For this reason, I used the statsmodel package initially to get good summary statistics about the model and to be able to understand if the assumptions of linear regression held. I did not look into prediction accuracy.
After understanding that multicollinearity was most likely an issue, I used Lasso Regression to try to reduce dimensionality (and understand important features in the model).
Finally, I used random forest to get a non-parametric understanding of the relationship and the important variables. I used k-fold cross-validation to make sure my sampling was robust.

### Ordinary Least Squares
* Pretty good Rsquare of .7
* However none of the features were statistically significant

### Lasso Regression
Training R^2: 0.6292522271697429\
Training RMSE: 12.017173642253404\
Test R^2: 0.1811440384154246\
Test RMSE: 16.509453527739193\
Training MAPE: 19.41%\
Test MAPE: 22.41%

-The level of prediction errors between the train and test set aren't huge, however, so I would say the model is not overfitting.
- Highlighted key features: truancy rates, mobility, and specific community impacts (e.g., Englewood and Austin neighborhoods had negative effects).

### Random Forest
Test RMSE: 15.623703298575046\
Test MAE: 11.917142857142858\
Test MAPE: 22.406299246613408\
Test R^2: 0.26665204577968527

Top Features: 
* Truancy (30% importance)
* mobility (23%)
* demographics (6%).

* Cross-validated Mean RMSE: 14.1710
* Standard Deviation of RMSE: 1.7558

<img width="846" alt="rf_top_10_featureImportance" src="https://github.com/user-attachments/assets/49bb55ed-3b72-4ae6-98f1-7ecfc4d70421" />

## Results and Discussion

The majority of our models yield an RMSE around 15. This means that the error of a prediction is generally 15 units (percentage points, since our target variable is a percentage... a value between 0 and 100) off. The standard deviation of our target is about 19, which means that the model performed pretty well (the error is less than sd).

**Truancy and Mobility Matter:** These are actionable metrics. Targeting interventions here may yield meaningful improvements in attainment.
Mobility rate measures how many students are transferring in and out of the school
Note: these are also interesting because they could point to home life or discipline/commitment issues. So may need interventions for things that are external to the schools

**Neighborhood Disparities Are Real:** Equity-focused policies could address systemic challenges in underperforming areas.
According to our linear regression output (where features were selected using lasso), some statistically significant contributors to the model were neighborhoods like Austin and Englewood, both known to be majority black, lower income, and have some struggles with crime.
However, we need to be careful while interpreting what those disparities mean. Our analytics show that some variables that intuitively should point to lower outcomes did not and vice versa (I’m thinking Facility Quality and Parent/Teacher partnerships)

**Unexpected Findings Need a Second Look:** The links between safety ratings, parental involvement, and attainment deserve further digging.

**Limitations of the Project**
Obviously taking a look at one academic school year is not enough to make any hardcore judgements about Chicago Public Schools. We would need a more comprehensive, wider set of data across time, more information about the schools themselves (size, demographics that are linked directly to the school and not just the census block group in which it resides, and any notable histories), and some domain knowledge to help us understand how useful our target metric is in terms of judging a school’s academic success).
But, the project does give us a good data-driven basis of where to start digging in further.

## Conclusion
**Factors of Academic Success**\
High rates of chronic truancy and student mobility were top predictors of lower reading attainment. These metrics alone explained a lot of the variation in school performance.
Schools in higher-income, predominantly white neighborhoods tended to perform better. But census data may not capture the full story of school demographics.

**Debunking Common Assumptions?**\
Weak safety ratings and poor parent-teacher partnership scores sometimes aligned with higher attainment. These counterintuitive findings might need deeper exploration.
Charter schools didn’t significantly outperform neighborhood schools on average, despite popular assumptions.

**Perhaps Schools are Teaching Something, Regardless of Overall Performance**\
8th-grade reading scores showed less variability than earlier grades and a higher median, hinting at a possible narrowing of performance gaps over time.

**What’s Next**
* Fine-Tune Interventions: This analysis suggests that programs addressing truancy and mobility could make a measurable impact. Further studies to understand these phenoms and how to address them may be in order.
* Expand the Data Scope: Larger datasets or alternative ways to measure neighborhood influence might provide richer insights. We could add in more progress report years to the dataset.
Communicate Findings: These insights could guide educators and policymakers to focus efforts where they’re needed most.

This project was a fascinating dive into what makes schools tick. The results raise as many questions as they answer, but it definitely lays a good data-driven basis for going deeper into some areas/issues that could yield real positive change.

