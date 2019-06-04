---
title: 'Education Dataset Analysis Pipeline: Walk Through #3'
output: html_document
---


```r
# Seed for random number generation
set.seed(42)
knitr::opts_chunk$set(cache.extra = knitr::rand_seed, eval = TRUE, echo = FALSE, results = 'hide',
                      message = FALSE, warning = FALSE)
```

# Background

One area of interest is the delivery of online instruction, which is becoming more prevalent: in 2007, over 3.9 million U.S. students were enrolled one or more online courses (Allen & Seaman, 2008). 

In this walkthrough, we examine the educational experiences of students in online science courses at a virtual middle school in order to characterize their motivation to achieve and their tangible engagement with the course in terms of behavioral trace measures. To do so, we use a robust data set, which includes self-reported motivation as well as behavioral trace data collected from a learning management system (LMS) to identify predictors of final course grade. Our work examines the idea of educational success in terms of student interactions with an online science course.

One meaningful perspective from which to consider students' engagement with online courses is related to their motivation to achieve. More specifically, it is important to consider how and why students are engaging with the course. Considering the psychological mechanisms behind achievement is valuable because doing so may help to identify meaningful points of intervention for educators and for researchers and administrators in online *and* face-to-face courses interested in the intersection between behavioral trace measures and students' motivational and affective experiences in such courses.

We take up the following four questions:

1. Is motivation more predictive of course grades as compared to other online indicators of engagement?
2. Which types of motivation are most predictive of achievement?
3. Which types of trace measures are most predictive of achievement?
4. How does a random forest compare to a simple linear model (regression)?

# Information about the dataset 

This dataset came from 499 students enrolled in online middle school science courses in 2015-2016. The data were originally collected for use as a part of a research study, though the findings have not been published anywhere, yet.

The setting of this study was a public, provider of individual online courses in a Midwestern state. In particular, the context was two semesters (Fall and Spring) of offerings of five online science courses (Anatomy & Physiology, Forensic Science, Oceanography, Physics, and Biology), with a total of 36 classes. 

Specific information in the dataset included:

- a pre-course survey students completed about their self-reported motivation in science — in particular, their perceived competence, utility value, and interest
- the time students spent on the course (obtained from the LMS, Blackboard) and their final course grades as well as their involvement in discussion forums
- for discussion board responses, we used the Linguistic Inquiry and Word Count (LIWC; Pennebaker, Boyd, Jordan, & Blackburn, 2015) to calculate the number of posts per student and variables for the mean levels of students' cognitive processing, positive affect, negative affect, and social-related discourse
---
title: 'Education Dataset Analysis Pipeline: Walk Through #3'
output: html_document
---



# Background

One area of interest is the delivery of online instruction, which is becoming more prevalent: in 2007, over 3.9 million U.S. students were enrolled one or more online courses (Allen & Seaman, 2008). 

In this walkthrough, we examine the educational experiences of students in online science courses at a virtual middle school in order to characterize their motivation to achieve and their tangible engagement with the course in terms of behavioral trace measures. To do so, we use a robust data set, which includes self-reported motivation as well as behavioral trace data collected from a learning management system (LMS) to identify predictors of final course grade. Our work examines the idea of educational success in terms of student interactions with an online science course.

One meaningful perspective from which to consider students' engagement with online courses is related to their motivation to achieve. More specifically, it is important to consider how and why students are engaging with the course. Considering the psychological mechanisms behind achievement is valuable because doing so may help to identify meaningful points of intervention for educators and for researchers and administrators in online *and* face-to-face courses interested in the intersection between behavioral trace measures and students' motivational and affective experiences in such courses.

We take up the following four questions:

1. Is motivation more predictive of course grades as compared to other online indicators of engagement?
2. Which types of motivation are most predictive of achievement?
3. Which types of trace measures are most predictive of achievement?
4. How does a random forest compare to a simple linear model (regression)?

# Information about the dataset 

This dataset came from 499 students enrolled in online middle school science courses in 2015-2016. The data were originally collected for use as a part of a research study, though the findings have not been published anywhere, yet.

The setting of this study was a public, provider of individual online courses in a Midwestern state. In particular, the context was two semesters (Fall and Spring) of offerings of five online science courses (Anatomy & Physiology, Forensic Science, Oceanography, Physics, and Biology), with a total of 36 classes. 

Specific information in the dataset included:

- a pre-course survey students completed about their self-reported motivation in science — in particular, their perceived competence, utility value, and interest
- the time students spent on the course (obtained from the LMS, Blackboard) and their final course grades as well as their involvement in discussion forums
- for discussion board responses, we used the Linguistic Inquiry and Word Count (LIWC; Pennebaker, Boyd, Jordan, & Blackburn, 2015) to calculate the number of posts per student and variables for the mean levels of students' cognitive processing, positive affect, negative affect, and social-related discourse

At the beginning of the semester, students were asked to complete the pre-course survey about their perceived competence, utility value, and interest. At the end of the semester, the time students spent on the course, their final course grades, and the contents of the discussion forums were collected.

In this walkthrough, we used the R package **caret** to carry out the analyses.

# Information on random forests

500 trees were grown as part of our random forest. We partitioned the data before conducting the main analysis so that neither the training nor the testing data set would be disproportionately representative of high-achieving or low-achieving students. The training data set consisted of 80% of the original data (n = 400 cases), whereas the testing data set consisted of 20% of the original data (n = 99 cases). We built our random forest model on the training data set, and then evaluated the model on the testing data set. Three variables were tried at each node.

Note that the random forest algorithm does not accept cases with missing data, and so we deleted cases listwise if data were missing. This decision eliminated 51 cases from our original data set, to bring us to our final sample size of 499 unique students.

For our analyses, we used Random Forest modeling (Breiman, 2001). Random forest is an extension of decision tree modeling, whereby a collection of decision trees are simultaneously "grown" and are evaluated based on out-of-sample predictive accuracy (Breiman, 2001).  Random forest is random in two main ways: first, each tree is only allowed to "see" and split on a limited number of predictors instead of all the predictors in the parameter space; second, a random subsample of the data is used to grow each individual tree, such that no individual case is weighted too heavily in the final prediction.

Whereas some machine learning approaches (e.g., boosted trees) would utilize an iterative model-building approach, random forest estimates all the decision trees at once. In this way, each tree is independent of every other tree. Thus, the random forest algorithm provides a robust regression approach that is distinct from other modeling approaches. The final random forest model aggregates the findings across all the separate trees in the forest in order to offer a collection of "most important" variables as well as a percent variance explained for the final model.

A random forest is well suited to the research questions that we had here because it allows for nonlinear modeling. We hypothesized complex relationships between students' motivation, their engagement with the online courses, and their achievement. For this reason, a traditional regressive or structural equation model would have been insufficient to model the parameter space we were interesting in modeling. Our random forest model had one outcome and eleven predictors. A common tuning parameter for machine learning models is the number of variables considered at each split (Kuhn, 2008); we considered three variables at each split for this analysis.  

The outcome was the final course grade that the student earned. The predictor variables included motivation variables (interest value, utility value, and science perceived competence) and trace variables (the amount of time spent in the course, the course name, the number of discussion board posts over the course of the semester, the mean level of cognitive processing evident in discussion board posts, the positive affect evident in discussion board posts, the negative affect evident in discussion board posts, and the social-related discourse evident in their discussion board posts). We used this random forest model to address all three of our research questions.

To interpret our findings, we examined three main things: (1) predictive accuracy of the random forest model, (2) variable importance, and (3) variance explained by the final random forest model.

# Analysis



First, we will load the data, *filter* the data to include only the data from one year, and *select* variables of interest.



## Use of caret

Here, we remove observations with missing data (per our note above about random forests requiring complete cases).



First, machine learning methods often involve using a large number of variables. Oftentimes, some of these variables will not be suitable to use: they may be highly correlated with other variables, for instance, or may have very little - or no - variability. Indeed, for the data set used in this study, one variables has the same (character string) value for all of the observations. We can detect this variable and any others using the following function:



If we look at `enrollment_status`, we will see that it is "Approved/Enrolled" for *all* of the students. When we use this in certian models, t may cause some problems, and so we remove it first.



Note that many times you may wish to pre-process the variables, such as by centering or scaling them; we could this with code like the following, which is not run here, as we will first try this out with the variables' original values.



We will want to make character string variables into factors.



Now, we will prepare the **train** and **test** datasets, using the caret function for creating data partitions. Here, the **p** argument specifies what proportion of the data we want to be in the **training** partition. Note that this function splits the data based upon the outcome, so that the training and test data sets will both have comparable values for the outcome. Note the `times = 1` argument; this function can be used to create *multiple* train and test sets, something we will describe in more detail later.



Finally, we will estimate the models.

Here, we will use the train function, passing *all* of the variables in the data frame (except for the outcome, or dependent variable, `final_grade`) as predictors. NOte that you can read more about the specific random forest implementation chosen [here](http://topepo.github.io/caret/train-models-by-tag.html#random-forest).


```
## Error: Required package is missing
```

```
## Error in eval(expr, envir, enclos): object 'rf_fit' not found
```

We have some results! First, we see that we have 400 samples, or 400 observations, the number in the train data set. No pre-processing steps were specified in the model fitting (note that) these the output of `preProcess` can be passed to `train()` to center, scale, and transform the data in many other ways. Next, note that a resampling technique has been used: this is not for validating the model (per se), but is rather for selecting tuning parameters, or options that need to be specified as a part of the modeling. These parameters can be manually provided, or can be estimated via strategies such as the bootstrap resample (or *k*-folds cross validation).

It appears that the model with the value of the **mtry** tuning parameter equal to 42 seemed to explain the data best, the **splirule* being "extratrees", and **min.node.size** held constant at a value of 5. 

Let's see if we end up with slightly different values if we change the resampling technique to cross-validation, instead of bootstrap resampling.





















