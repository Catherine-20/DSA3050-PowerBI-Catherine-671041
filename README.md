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

At least 8 significant transformations were carried out, documented below as
Problem -> Transformation -> Reason -> Result.

**1. Aggregating studentVle.csv (10+ million rows)**
Problem: studentVle.csv contained over 10 million rows of daily click-level
VLE interaction data - far too large and granular to load directly into
Power BI or to be analytically useful at that grain.
Transformation: Removed the id_site column, then used Group By to aggregate
on code_module, code_presentation, and id_student, summing sum_click into a
new TotalClicks column.
Reason: Reduces the data to a meaningful analytical grain (engagement per
student per course) while making the file practical to load and model.
Result: A new query, FactVLEEngagement, with one row per student per course
presentation and a TotalClicks measure.

**2. Handling missing imd_band values**
Problem: The imd_band field (deprivation index) in studentInfo contained
blank/null values for some students.
Transformation: Used Replace Values to replace blank imd_band entries with
"Unknown".
Reason: Leaving nulls in a categorical field causes them to be excluded or
misrepresented in visuals; an explicit "Unknown" category preserves the row
while making the missingness

## Section C: Data Modelling

### Fact tables
Two fact tables were used rather than one large flat table:

- **FactAssessment** (from studentAssessment.csv, merged with assessments.csv):
  contains one row per assessment attempt (score, submission date, whether the
  score was banked/carried over). This is the natural transactional grain of
  the dataset - each row is a real event (a student submitting or being scored
  on an assessment).
- **FactVLEEngagement** (aggregated from studentVle.csv): contains one row per
  student per course presentation, with total VLE clicks and final course
  result. The raw studentVle.csv contained 10+ million daily click records,
  which was too large to load meaningfully into Power BI, so it was
  aggregated in Power Query (Group By: code_module, code_presentation,
  id_student, summing sum_click) before loading. This is documented in
  Section B.

Using two fact tables at different grains (assessment-level vs
student-course-level) allowed both a fine-grained view of performance and a
broader view of engagement, sharing the same dimensions.

### Dimension tables
- **DimStudent**: one row per unique student (gender, region, highest
  education, IMD band, age band, disability). Built by referencing
  studentInfo and removing duplicates on id_student only, since the source
  table originally had one row per student per course.
- **DimCourse**: one row per course module/presentation (code_module,
  code_presentation, module length), with a concatenated CourseKey
  (code_module & "_" & code_presentation) added because code_module alone
  is not unique - the same module (e.g. "AAA") repeats across multiple
  presentations (years/semesters).
- **DimDate**: a small custom date dimension representing the four course
  presentation periods in the dataset (2013B, 2013J, 2014B, 2014J), since
  the raw data does not contain calendar dates, only day-offsets and
  presentation codes.

### Relationships
- DimStudent to FactVLEEngagement: one-to-many, single direction, on id_student
- DimStudent to FactAssessment: one-to-many, single direction, on id_student
- DimCourse to FactVLEEngagement: one-to-many, single direction, on CourseKey
- DimCourse to FactAssessment: one-to-many, single direction, on CourseKey
- DimDate to FactVLEEngagement: one-to-many, single direction, on
  PresentationCode / code_presentation

All relationships are one-to-many with a single cross-filter direction, so
each dimension filters its connected fact tables without creating ambiguous
or bidirectional filter paths.

### Modelling challenges encountered
- code_module alone could not be used as a relationship key for DimCourse,
  since the same module code repeats across multiple presentations. This was
  resolved by creating a concatenated CourseKey column in both DimCourse and
  the fact tables.
- FactAssessment does not include code_presentation, so it was not connected
  to DimDate. Only FactVLEEngagement is connected to DimDate. This was a
  deliberate scope decision rather than an oversight: FactVLEEngagement
  already supports the time-based (presentation-period) analysis needed for
  this project, so a full merge to add code_presentation into FactAssessment
  was not repeated.

## Section D: DAX & Business Calculations

12 DAX measures were created across three levels of complexity. Six of the
most important are explained below.

### 1. Total Students
`Total Students = DISTINCTCOUNT(FactAssessment[id_student])`
Calculates the number of unique students represented in the assessment
data. This is used as the base for several rate calculations (e.g. Pass
Rate %) and gives management an immediate sense of scale on the Executive
Overview page. DISTINCTCOUNT is used rather than COUNTROWS because a
student can appear multiple times (once per assessment).

### 2. Pass Rate %
`Pass Rate % = DIVIDE([Pass Count], [Total Students], 0)`
Calculates the proportion of students whose final result was "Pass",
relative to the total number of students. DIVIDE is used instead of the /
operator to safely return 0 rather than an error when the denominator is
zero (e.g. in an empty filter context). This measure is central to the
Executive Overview page as a headline KPI, and changes dynamically based on
whichever module, presentation, or demographic slicer is applied.

### 3. Average Clicks per Student
`Average Clicks per Student = DIVIDE([Total Clicks], DISTINCTCOUNT(FactVLEEngagement[id_student]), 0)`
Calculates average VLE engagement per student, used to investigate whether
engagement level relates to outcome. This measure responds to filter
context - for example, filtering to only "Withdrawn" students shows their
average engagement compared to the whole cohort. Used on the Advanced/
Diagnostic Analysis page.

### 4. Distinction Rate %

## Section E: Dashboards
## Section E: Professional Power BI Dashboards

Three report pages were built, moving from a high-level overview to
progressively deeper analysis:

### Page 1: Executive Overview
KPI cards for Total Students, Pass Rate %, Withdrawal Rate %, and Average
Clicks per Student give an immediate summary of performance. A column
chart breaks down final results (Pass/Fail/Withdrawn/Distinction), and a
donut chart shows student distribution across modules. Module and
presentation-year slicers let a manager quickly narrow the view.

### Page 2: Detailed Analysis
A sortable table compares Pass Rate %, Distinction Rate %, and Withdrawal
Rate % across every module. A column chart breaks down Pass Rate % by age
band to surface demographic performance differences. A scatter chart plots
Average Clicks per Student against Average Score by module, visually
testing whether engagement relates to performance. An IMD band slicer
allows filtering by deprivation index across the whole page.

### Page 3: Advanced/Diagnostic Analysis
This page investigates why outcomes occur rather than just what happened.
A table cross-references IMD band and age band against Withdrawal Rate %
and Average Clicks per Student, to see where withdrawal concentrates. A
stacked column chart shows RegistrationStatus by module. A bar chart
compares Pass Rate % between students flagged as "Above Average" vs "Below
Average" engagement (using the High Engagement Flag measure), directly
testing the relationship between engagement and outcome. A ranking table
lists students by Score Rank to identify top and bottom performers.

### Interactivity
All three pages use slicers (module, presentation year, IMD band) that
cross-filter every visual on their page. Visuals share the model's
relationships, so selecting a module or demographic group in one visual
filters related visuals automatically.

## Section F: Repository Notes
*(to be completed)*
