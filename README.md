# DSA 3050A – Business Intelligence & Data Visualization
## End Semester Practical Examination

**Student:** Catherine
**Registration No.:** 671041
**Software:** Microsoft Power BI Desktop

---

## Section A: Dataset Selection & Understanding

### 1. Source of the dataset
Open University Learning Analytics Dataset (OULAD), accessed via Kaggle
(https://www.kaggle.com/datasets/anlgrbz/student-demographics-online-education-dataoulad),
originally published by The Open University (UK) at analyse.kmi.open.ac.uk.

### 2. What the dataset represents
The dataset contains anonymised data on approximately 32,600 students across
7 undergraduate course modules and multiple course presentations (semesters).
It covers student demographics, module registration, assessment scores, final
course outcomes, and student interaction (click activity) with the university's
Virtual Learning Environment (VLE).

The data is split across seven related tables:
- courses.csv – list of modules and presentations
- studentInfo.csv – student demographics and final result
- studentRegistration.csv – registration and unregistration dates
- assessments.csv – details of each assessment per module
- studentAssessment.csv – individual student scores per assessment
- vle.csv – VLE activity/resource metadata
- studentVle.csv – daily student click interactions with the VLE

### 3. Why this dataset was selected
The dataset is real, relational, and not pre-cleaned or pre-summarized, requiring
genuine data preparation decisions rather than simple visualization of already-clean
data. It is large enough to support meaningful BI analysis, contains a clear
business problem relevant to education (student performance and retention), and
offers multiple dimensions (demographic, temporal, behavioural) for analysis.

### 4. Main variables available
- Demographics: gender, age band, region, highest education level, IMD band
  (deprivation index), disability status
- Course information: module code, presentation code (year + semester), module length
- Registration: date of registration, date of unregistration (if withdrawn)
- Assessment: assessment type, score, weighting, submission date
- Outcome: final_result (Pass / Fail / Withdrawn / Distinction)
- Engagement: VLE click activity by date and activity type

### 5. Business/analytical problem
What factors are associated with student underperformance and withdrawal, and
can demographic or engagement patterns help identify at-risk students earlier
in a course?

### 6. Analytical questions the Power BI solution should answer
1. What is the pass/fail/withdrawal/distinction rate across different course
   modules and presentations?
2. Does prior educational attainment or deprivation index (IMD band) correlate
   with final outcome?
3. Is there a relationship between VLE engagement (click activity) and final result?
4. Do students who unregister early differ in outcome patterns from those who
   complete the course?
5. Which demographic groups (age band, gender, disability, region) show the
   strongest or weakest performance trends?

---

## Section B: Power Query – Data Cleaning & Transformation
*(to be completed)*

## Section C: Data Modelling
*(to be completed)*

## Section D: DAX & Business Calculations
*(to be completed)*

## Section E: Dashboards
*(to be completed)*

## Section F: Repository Notes
*(to be completed)*
