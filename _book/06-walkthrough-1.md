---
title: 'Education Dataset Analysis Pipeline: Walkthrough #1'
output: html_document
---



# Background 

In the 2015-2016 and 2016-2017 school years, researchers at Michigan State University carried out a study on students' motivation to learn in online science classes. The online science classes were part of a statewide online course provider designed to supplement (and not replace) students' enrollment in their local school. For example, students may choose to enroll in an online physics class because one was not offered at their school (or they were not able to take it given their schedule).

The study involved a number of different data sources which were brought to bear to understand students' motivation:

1. A self-report survey for three distinct but related aspects of students' motivation
2. Log-trace data, such as data output from the learning management system
3. Achievement-related and gradebook data
4. Discussion board data
5. Achievement-related (i.e., final grade) data

First, these different data sources will be described in terms of how they were provided by the school.

## 1. Self-report survey 

This was data collected before the start of the course via self-report survey. The survey included 10 items, each corresponding to one of three *measures*, namely, for interest, utility value, and perceived competence:

1.	I think this course is an interesting subject. (Interest)
2.	What I am learning in this class is relevant to my life. (Utility value)
3.	I consider this topic to be one of my best subjects. (Perceive competence)
4.	I am not interested in this course. (Interest - reverse coded)
5.	I think I will like learning about this topic. (Interest)
6.	I think what we are studying in this course is useful for me to know. (Utility value)
7.	I don’t feel comfortable when it comes to answering questions in this area. (Perceived competence)
8.	I think this subject is interesting. (Interest)
9.	I find the content of this course to be personally meaningful. (Utility value)
10.	I’ve always wanted to learn more about this subject. (Interest)

## 2. Log-trace data 

Log-trace data is data generated from our interactions with digital technologies. In education, an increasingly common source of log-trace data is that generated from interactions with learning management systems. The data for this walk-through is a *summary of* log-trace data, namely, the number of minutes students spent on the course. Thus, while this data is rich, you can imagine even more complex sources of log-trace data (i.e. timestamps associated with when students started and stopped accessing the course!).

## 3. Achievement-related and gradebook data

This is a common source of data, namely, one associated with graded assignments students completed. In this walkthrough, we just examine students' final grade.

## 4. Discussion board data
<!-- NOTE - may not include this, as it is hard to confidently anonymize a medium-ish sized data set -->

Discussion board data is both rich and unstructured, in that it is primarily in the form of written text. We examine a small subset of the discussion board data in this walkthrough.

# Processing the data


```r
library(readxl)
library(tidyverse)
library(lubridate)
library(here)
```


```r
# Gradebook and log-trace data for F15 and S16 semesters
s12_course_data <- read_csv(
  here(
    "data", 
    "online-science-motivation", 
    "raw", 
    "s12-course-data.csv"
  )
)

# Pre-survey for the F15 and S16 semesters
s12_pre_survey  <- read_csv(
  here(
    "data", 
    "online-science-motivation", 
    "raw", 
    "s12-pre-survey.csv"
  )
) 

# Log-trace data for F15 and S16 semesters - ts is for time spent
s12_time_spent <- read_csv(
  here(
    "data", 
    "online-science-motivation", 
    "raw", 
    "s12-course-minutes.csv"
  )
)
```

## Viewing the data


```r
s12_pre_survey 
```

```
## # A tibble: 1,102 x 16
##    RespondentId StartDate CompletedDate LanguageCode opdata_CourseID
##           <dbl> <chr>     <chr>         <chr>        <chr>          
##  1       426746 2015.08.… <NA>          en           FrScA-S116-01  
##  2       426775 2015.08.… 2015.08.24 1… en           BioA-S116-01   
##  3       427483 2015.08.… <NA>          en           OcnA-S116-03   
##  4       429883 2015.09.… 2015.09.02 1… en           AnPhA-S116-01  
##  5       430158 2015.09.… 2015.09.03 9… en           AnPhA-S116-01  
##  6       430161 2015.09.… 2015.09.03 9… en           AnPhA-S116-02  
##  7       430162 2015.09.… 2015.09.03 9… en           AnPhA-T116-01  
##  8       430167 2015.09.… 2015.09.03 9… en           BioA-S116-01   
##  9       430170 2015.09.… 2015.09.03 9… en           BioA-T116-01   
## 10       430172 2015.09.… 2015.09.03 9… en           PhysA-S116-01  
## # … with 1,092 more rows, and 11 more variables: opdata_username <chr>,
## #   Q1MaincellgroupRow1 <dbl>, Q1MaincellgroupRow2 <dbl>,
## #   Q1MaincellgroupRow3 <dbl>, Q1MaincellgroupRow4 <dbl>,
## #   Q1MaincellgroupRow5 <dbl>, Q1MaincellgroupRow6 <dbl>,
## #   Q1MaincellgroupRow7 <dbl>, Q1MaincellgroupRow8 <dbl>,
## #   Q1MaincellgroupRow9 <dbl>, Q1MaincellgroupRow10 <dbl>
```

```r
s12_course_data
```

```
## # A tibble: 29,711 x 16
##    CourseSectionOr… Bb_UserPK EnrollmentStatus EnrollmentReason Gender
##    <chr>                <dbl> <chr>            <chr>            <chr> 
##  1 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  2 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  3 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  4 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  5 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  6 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  7 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  8 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
##  9 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
## 10 AnPhA-S116-01        60186 Approved/Enroll… Course Unavaila… M     
## # … with 29,701 more rows, and 11 more variables: FinalGradeCEMS <dbl>,
## #   Gradebook_Item <chr>, Item_Position <dbl>, Gradebook_Type <chr>,
## #   Gradebook_Date <chr>, Grade_Category <chr>, Status <lgl>,
## #   Points_Earned <chr>, Points_Attempted <dbl>, Points_Possible <dbl>,
## #   last_access_date <drtn>
```

```r
s12_time_spent
```

```
## # A tibble: 598 x 6
##    CourseID CourseSectionID CourseSectionOrigID Bb_UserPK   CUPK TimeSpent
##       <dbl>           <dbl> <chr>                   <dbl>  <dbl>     <dbl>
##  1       27           17146 OcnA-S116-01            44638 190682     1383.
##  2       27           17146 OcnA-S116-01            54346 194259     1191.
##  3       27           17146 OcnA-S116-01            57981 196014     3343.
##  4       27           17146 OcnA-S116-01            66740 190463      965.
##  5       27           17146 OcnA-S116-01            67920 191593     4095.
##  6       27           17146 OcnA-S116-01            85355 190104      595.
##  7       27           17146 OcnA-S116-01            85644 190685     1632.
##  8       27           17146 OcnA-S116-01            86349 191713     1601.
##  9       27           17146 OcnA-S116-01            86460 191887     1891.
## 10       27           17146 OcnA-S116-01            87970 194256     3123.
## # … with 588 more rows
```

## Processing the pre-survey data

Often, survey data needs to be processed in order to be (most) useful. Here, we process the self-report items into three scales, for: interest, self-efficacy, and utility value. We do this by 

 - Renaming the question variables to something more managable 
 - Reversing the response scales on questions 4 and 7 
 - Categorizing each question into a measure 
 - Computing the mean of each measure 


```r
s12_pre_survey  <- s12_pre_survey  %>%
    # Rename the qustions something easier to work with because R is case sensitive
    # and working with variable names in mix case is prone to error
    rename(q1 = Q1MaincellgroupRow1,
           q2 = Q1MaincellgroupRow2,
           q3 = Q1MaincellgroupRow3,
           q4 = Q1MaincellgroupRow4,
           q5 = Q1MaincellgroupRow5,
           q6 = Q1MaincellgroupRow6,
           q7 = Q1MaincellgroupRow7,
           q8 = Q1MaincellgroupRow8,
           q9 = Q1MaincellgroupRow9,
           q10 = Q1MaincellgroupRow10) %>% 
    # Convert all question responses to numeric
    mutate_at(vars(q1:q10), funs(as.numeric))

# Function for reversing scales 
reverse_scale <- function(question) {
    # Reverses the response scales for consistency 
    #   Args: 
    #     question: survey question 
    #   Returns: a numeric converted response
  # Note: even though 3 is not transformed, case_when expects a match for all
  # possible conditions, so it's best practice to label each possible input
  # and use TRUE ~ as the final statement returning NA for unexpected inputs
  x <- case_when(question == 1 ~ 5, 
                 question == 2 ~ 4,
                 question == 4 ~ 2,
                 question == 5 ~ 1,
                 question == 3 ~ 3,
                 TRUE ~ NA_real_)
  x
}

# Reverse scale for questions 4 and 7
s12_pre_survey <- s12_pre_survey %>%
  mutate(q4 = reverse_scale(q4),
         q7 = reverse_scale(q7))

# Add measure variable 
s12_measure_mean <- s12_pre_survey %>% 
    # Gather questions and responses 
    gather(question, response, c(q1:q10)) %>% 
    mutate(
        measure = case_when(
            question %in% c("q1", "q4", "q5", "q8", "q10") ~ "int", 
            question %in% c("q2", "q6", "q9") ~ "uv", 
            question %in% c("q3", "q7") ~ "pc", 
            TRUE ~ NA_character_
        )) %>% 
    group_by(measure) %>% 
    summarise(
        # Mean response for each measure
        mean_response = mean(response, na.rm = TRUE), 
        # Percent of each measure that had NAs in the response field
        percent_NA = mean(is.na(response))
        )

s12_measure_mean
```

```
## # A tibble: 3 x 3
##   measure mean_response percent_NA
##   <chr>           <dbl>      <dbl>
## 1 int              4.26      0.171
## 2 pc               3.65      0.170
## 3 uv               3.76      0.170
```

## Processing the course data

We also can process the course data in order to obtain more information.


```r
# split course section into components
s12_course_data <- s12_course_data %>%
  separate(col = CourseSectionOrigId,
           into = c('subject', 'semester', 'section'),
           sep = '-',
           remove = FALSE)
```

```
## Error in eval_tidy(enquo(var), var_env): object 'CourseSectionOrigId' not found
```

This led to pulling out the subject, semester, and section from the course ID; variables that we can use later on.

## Joining the data

To join the course data and pre-survey data, we need to create similar *keys*. In other words, our goal here is to have one variable that matches across both datasets, so that we can merge the datasets on the basis of that variable. 

For these data, both have variables for the course and the student, though they have different names in each. Our first goal will be to rename two variables in each of our datasets so that they will match. One variable will correspond to the course, and the other will correspond to the student. We are not changing anything in the data itself at this step - instead, we are just cleaning it up so that we can look at the data all in one place.

Let's start with the pre-survey data. We will rename RespondentID and opdata_CourseID to be student_id and course_id, respectively.


```r
s12_pre_survey <- s12_pre_survey %>% 
    rename(student_id = RespondentId,
           course_id = opdata_CourseID)

s12_pre_survey
```

```
## # A tibble: 1,102 x 16
##    student_id StartDate CompletedDate LanguageCode course_id
##         <dbl> <chr>     <chr>         <chr>        <chr>    
##  1     426746 2015.08.… <NA>          en           FrScA-S1…
##  2     426775 2015.08.… 2015.08.24 1… en           BioA-S11…
##  3     427483 2015.08.… <NA>          en           OcnA-S11…
##  4     429883 2015.09.… 2015.09.02 1… en           AnPhA-S1…
##  5     430158 2015.09.… 2015.09.03 9… en           AnPhA-S1…
##  6     430161 2015.09.… 2015.09.03 9… en           AnPhA-S1…
##  7     430162 2015.09.… 2015.09.03 9… en           AnPhA-T1…
##  8     430167 2015.09.… 2015.09.03 9… en           BioA-S11…
##  9     430170 2015.09.… 2015.09.03 9… en           BioA-T11…
## 10     430172 2015.09.… 2015.09.03 9… en           PhysA-S1…
## # … with 1,092 more rows, and 11 more variables: opdata_username <chr>,
## #   q1 <dbl>, q2 <dbl>, q3 <dbl>, q4 <dbl>, q5 <dbl>, q6 <dbl>, q7 <dbl>,
## #   q8 <dbl>, q9 <dbl>, q10 <dbl>
```

Looks better now!

Let's proceed to the course data. Our goal is to rename two variables that correspond to the course and the student so that we can match with the other variables we just created for the pre-survey data.


```r
s12_course_data <- s12_course_data %>% 
    rename(student_id = Bb_UserPK,
           course_id = CourseSectionOrigID)

s12_course_data
```

```
## # A tibble: 29,711 x 16
##    course_id student_id EnrollmentStatus EnrollmentReason Gender
##    <chr>          <dbl> <chr>            <chr>            <chr> 
##  1 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  2 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  3 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  4 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  5 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  6 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  7 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  8 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  9 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
## 10 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
## # … with 29,701 more rows, and 11 more variables: FinalGradeCEMS <dbl>,
## #   Gradebook_Item <chr>, Item_Position <dbl>, Gradebook_Type <chr>,
## #   Gradebook_Date <chr>, Grade_Category <chr>, Status <lgl>,
## #   Points_Earned <chr>, Points_Attempted <dbl>, Points_Possible <dbl>,
## #   last_access_date <drtn>
```

Now that we have two variables that are consistent across both datasets - we have called them "course_id" and "student_id" -  we can join these using the **dplyr** function, `left_join()`. 
Let's save our joined data as a new object called "dat."


```r
dat <- left_join(s12_course_data, 
                 s12_pre_survey, 
                 by = c("student_id", "course_id"))

dat
```

```
## # A tibble: 29,711 x 30
##    course_id student_id EnrollmentStatus EnrollmentReason Gender
##    <chr>          <dbl> <chr>            <chr>            <chr> 
##  1 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  2 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  3 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  4 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  5 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  6 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  7 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  8 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  9 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
## 10 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
## # … with 29,701 more rows, and 25 more variables: FinalGradeCEMS <dbl>,
## #   Gradebook_Item <chr>, Item_Position <dbl>, Gradebook_Type <chr>,
## #   Gradebook_Date <chr>, Grade_Category <chr>, Status <lgl>,
## #   Points_Earned <chr>, Points_Attempted <dbl>, Points_Possible <dbl>,
## #   last_access_date <drtn>, StartDate <chr>, CompletedDate <chr>,
## #   LanguageCode <chr>, opdata_username <chr>, q1 <dbl>, q2 <dbl>,
## #   q3 <dbl>, q4 <dbl>, q5 <dbl>, q6 <dbl>, q7 <dbl>, q8 <dbl>, q9 <dbl>,
## #   q10 <dbl>
```

Just one more data frame to merge:


```r
s12_time_spent <- s12_time_spent %>%
  rename(student_id = Bb_UserPK, 
         course_id = CourseSectionOrigID)
s12_time_spent <- s12_time_spent %>%
  mutate(student_id = as.integer(student_id))
dat <- dat %>% 
  left_join(s12_time_spent, 
            by = c("student_id", "course_id"))
```

Note that they're now combined, even though the course data has many more rows: The pre_survey data has been joined for each student by course combination.

We have a pretty large data frame! Let's take a quick look.


```r
dat
```

```
## # A tibble: 29,711 x 34
##    course_id student_id EnrollmentStatus EnrollmentReason Gender
##    <chr>          <dbl> <chr>            <chr>            <chr> 
##  1 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  2 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  3 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  4 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  5 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  6 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  7 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  8 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
##  9 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
## 10 AnPhA-S1…      60186 Approved/Enroll… Course Unavaila… M     
## # … with 29,701 more rows, and 29 more variables: FinalGradeCEMS <dbl>,
## #   Gradebook_Item <chr>, Item_Position <dbl>, Gradebook_Type <chr>,
## #   Gradebook_Date <chr>, Grade_Category <chr>, Status <lgl>,
## #   Points_Earned <chr>, Points_Attempted <dbl>, Points_Possible <dbl>,
## #   last_access_date <drtn>, StartDate <chr>, CompletedDate <chr>,
## #   LanguageCode <chr>, opdata_username <chr>, q1 <dbl>, q2 <dbl>,
## #   q3 <dbl>, q4 <dbl>, q5 <dbl>, q6 <dbl>, q7 <dbl>, q8 <dbl>, q9 <dbl>,
## #   q10 <dbl>, CourseID <dbl>, CourseSectionID <dbl>, CUPK <dbl>,
## #   TimeSpent <dbl>
```

It looks like we have neary 30,000 observations from 30 variables.

Now that our data are ready to go, we can start to ask some questions of the data. 

# Visualizations and Models

One thing we might be wondering is how time spent on course is related to students' final grade. Let's first calculate the percentage of points students earned as a measure of their final grade (noting that the teacher may have assigned a different grade--or weighted their grades in ways not reflected through the points).


```r
dat <- dat %>% 
    group_by(student_id, course_id) %>% 
    mutate(Points_Earned = as.integer(Points_Earned)) %>% 
    summarize(total_points_possible = sum(Points_Possible, na.rm = TRUE),
              total_points_earned = sum(Points_Earned, na.rm = TRUE)) %>% 
    mutate(percentage_earned = total_points_earned/total_points_possible) %>% 
    ungroup() %>% 
    left_join(dat) # note that we join this back to the original data frame to retain all of the variables
```

## Visualization of the relationship between time spent on course and percentage of points earned

<!-- This is really trivial and obvious; need a new/better relationship -->


```r
ggplot(dat, aes(x = TimeSpent, y = percentage_earned)) +
    geom_point()
```

<img src="06-walkthrough-1_files/figure-html/unnamed-chunk-11-1.png" width="672" />

There appears to be *some* relationship. What if we added a line of best fit - a linear model?


```r
ggplot(dat, aes(x = TimeSpent, y = percentage_earned)) +
    geom_point() + 
    geom_smooth(method = "lm")
```

<img src="06-walkthrough-1_files/figure-html/unnamed-chunk-12-1.png" width="672" />

So, it appeares that the more time students spent on the course, the more points they earned.

# Linear model (regression)

We can find out exactly what the relationship is using a linear model.


```r
m_linear <- lm(percentage_earned ~ TimeSpent, data = dat)
summary(m_linear)
```

```
## 
## Call:
## lm(formula = percentage_earned ~ TimeSpent, data = dat)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.63001 -0.07894  0.05366  0.15742  0.34544 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 6.102e-01  2.158e-03  282.77   <2e-16 ***
## TimeSpent   7.983e-05  9.399e-07   84.94   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.2236 on 29481 degrees of freedom
##   (228 observations deleted due to missingness)
## Multiple R-squared:  0.1966,	Adjusted R-squared:  0.1966 
## F-statistic:  7214 on 1 and 29481 DF,  p-value: < 2.2e-16
```

## But what about different courses?

Is there course-specific differences in how much time students spend on the 
course as well as in how time spent is related to the percentage of points 
students earned?


```r
ggplot(dat, aes(x = TimeSpent, y = percentage_earned, color = course_id)) +
    geom_point()
```

<img src="06-walkthrough-1_files/figure-html/unnamed-chunk-14-1.png" width="672" />


```r
ggplot(dat, aes(x = TimeSpent, y = percentage_earned, color = course_id)) +
    geom_point() +
    geom_smooth(method = "lm")
```

<img src="06-walkthrough-1_files/figure-html/unnamed-chunk-15-1.png" width="672" />

There appears to be so. One way we can test is to use what is called a multi-level model. This requires a new package; one of the most common for estimating these types of models is **lme4**. We use it very similarly to the `lm()` function, but we pass it an additional argument about what the *groups*, or levels, in the data are.


```r
# install.packages("lme4")
library(lme4)
m_course <- lmer(percentage_earned ~ TimeSpent + (1|course_id), data = dat)
summary(m_course)
```

```
## Linear mixed model fit by REML ['lmerMod']
## Formula: percentage_earned ~ TimeSpent + (1 | course_id)
##    Data: dat
## 
## REML criterion at convergence: -8619.3
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -3.3371 -0.3701  0.2154  0.6418  2.1217 
## 
## Random effects:
##  Groups    Name        Variance Std.Dev.
##  course_id (Intercept) 0.008815 0.09389 
##  Residual              0.043475 0.20851 
## Number of obs: 29483, groups:  course_id, 26
## 
## Fixed effects:
##              Estimate Std. Error t value
## (Intercept) 5.820e-01  1.863e-02   31.25
## TimeSpent   8.884e-05  9.273e-07   95.81
## 
## Correlation of Fixed Effects:
##           (Intr)
## TimeSpent -0.090
## fit warnings:
## Some predictor variables are on very different scales: consider rescaling
```

A common way to understand how much variability is at the group level is to calculate the *intra-class* correlation. This value is the proportion of the variability in the outcome (the *y*-variable) that is accounted for solely by the groups identified in the model. There is a useful function in the **sjstats** package for doing this.


```r
# install.packages("sjstats")
library(sjstats)
icc(m_course)
```

```
## # Intraclass Correlation Coefficient
## 
##      Adjusted ICC: 0.169
##   Conditional ICC: 0.131
```

This shows that nearly 17% of the variability in the percentage of points students earned can be explained simply by knowing what class they are in. 
