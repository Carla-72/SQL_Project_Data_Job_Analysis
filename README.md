# Introduction
Focusing on Data Analyst roles, this project explores top-paying jobs, in-demand skills, and where high demand meets high salary in data analytics.
# Background
The motivation behind this project stemmed from my desire to understand the data analyst job market better. I aimed to discover which skills are paid the most and in demand, supporting job seekers in a more targeted and effective manner.
The data used for this project comes from [Luke Barousse's SQL Course](/https://lukebarousse.com/sql). It has detailed insights on job titles, salaries, locations, and essential skills.

### The questions I want to answer through this project of SQL queries are:
1. What are the top-paying data analyst jobs?
2. What skills are required for the top-paying data analyst roles?
3. What are the most in-demand skills for data analysts?
4. What are the top skills based on salary?
5. What are the most optimal skills to learn, meaning they are highly demanded and high-paying?

# Tools I Used
For this project, I harnessed the power of several key tools:
- **SQL:** The backbone of the analysis, allowing me to query the database and extract critical insights.
- **PostgreSQL:** The chosen database management system, ideal for handling the job posting data.
- **Visual Studio Code:** My go-to source-code editor for database management and executing SQL queries.
- **Git and GitHub:** Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.

# The Analysis
Each query for this project aimed at investigating specific aspects of the data analyst job market. Here’s how I approached each question:

### 1. Top-Paying Data Analyst Jobs
To identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on jobs in the UAE. This query highlights the high paying opportunities in the field.
```sql
SELECT
    job_id,
    job_title,
    company_dim.name AS company_name,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date

FROM job_postings_fact
LEFT JOIN company_dim on job_postings_fact.company_id = company_dim.company_id

WHERE 
    job_title_short = 'Data Analyst'
    AND job_country = 'United Arab Emirates'
    AND salary_year_avg IS NOT NULL

ORDER BY salary_year_avg DESC
LIMIT 10;
```

| Job ID  | Job Title                    | Company Name        | Location                         | Schedule  | Avg Yearly Salary (USD) | Posted Date           |
|--------:|------------------------------|---------------------|----------------------------------|-----------|-------------------------|-----------------------|
| 924957  | Data Analyst                 | invygo              | Dubai - United Arab Emirates     | Full-time | 111,175                 | 2023-08-01 16:33:29   |
| 374696  | Data Analyst                 | Bayut \| dubizzle   | Dubai - United Arab Emirates     | Full-time | 98,500                  | 2023-03-10 17:47:17   |
| 1040368 | Decision & Data Analyst-UAE  | Foodics             | Dubai - United Arab Emirates     | Full-time | 98,500                  | 2023-03-27 15:35:58   |
| 387001  | Product Data Analyst         | Binance.US          | Dubai - United Arab Emirates     | Full-time | 79,200                  | 2023-01-20 08:23:33   |
| 75121   | Product Data Analyst         | Sana Commerce       | Dubai - United Arab Emirates     | Full-time | 72,000                  | 2023-08-11 06:29:52   |


### 2. Skills for Top Paying Jobs
To understand what skills are required for the top-paying jobs, I joined the job postings with the skills data, providing insights into what employers value for high-compensation roles.
```sql
WITH top_paying_jobs AS 
(
    SELECT
        job_id,
        job_title,
        company_dim.name AS company_name,
        salary_year_avg
        
    FROM job_postings_fact
    LEFT JOIN company_dim on job_postings_fact.company_id = company_dim.company_id

    WHERE 
        job_title_short = 'Data Analyst'
        AND job_country = 'United Arab Emirates'
        AND salary_year_avg IS NOT NULL

    ORDER BY salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim on top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id

ORDER BY
    salary_year_avg DESC;
```

| Job ID  | Job Title                     | Company Name        | Avg Yearly Salary (USD) | Skills |
|--------:|-------------------------------|---------------------|-------------------------|--------|
| 924957  | Data Analyst                  | invygo              | 111,175                 | SQL, Python, R, Tableau, Power BI |
| 374696  | Data Analyst                  | Bayut \| dubizzle   | 98,500                  | SQL |
| 1040368 | Decision & Data Analyst-UAE   | Foodics             | 98,500                  | SQL, Python, R, Tableau, Power BI, Looker |
| 387001  | Product Data Analyst          | Binance.US          | 79,200                  | SQL, Python, R, Tableau |
| 75121   | Product Data Analyst          | Sana Commerce       | 72,000                  | SQL, Python, AWS, Excel, Tableau, Power BI, PowerPoint |


### 3. In-Demand Skills for Data Analysts
This query helped identify the skills most frequently requested in UAE job postings, directing focus to areas with high demand.
```sql
SELECT
    skills,
    COUNT(job_postings_fact.job_id) AS demand_count

FROM job_postings_fact
INNER JOIN skills_job_dim on job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id

WHERE 
    job_title_short = 'Data Analyst'
    AND job_country = 'United Arab Emirates'

GROUP BY skills

ORDER BY
    demand_count DESC
LIMIT 5;
```

| Skill     | Demand Count |
|-----------|--------------|
| SQL       | 1,009        |
| Excel     | 812          |
| Python    | 608          |
| Tableau   | 562          |
| Power BI  | 518          |


### 4. Skills Based on Salary
Exploring the average salaries associated with different skills revealed which skills are the highest paying.
```sql
SELECT
    skills,
    ROUND(AVG(job_postings_fact.salary_year_avg),1) AS avg_salary

FROM job_postings_fact
INNER JOIN skills_job_dim on job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id

WHERE 
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_country = 'United Arab Emirates'

GROUP BY skills

ORDER BY
   avg_salary DESC
LIMIT 25;
```

| Skill       | Avg Yearly Salary (USD) |
|-------------|-------------------|
| Looker      | 98,500            |
| R           | 96,291.7          |
| Power BI    | 93,891.7          |
| SQL         | 91,875            |
| Tableau     | 90,218.8          |
| Python      | 90,218.8          |
| PowerPoint  | 72,000            |
| Excel       | 72,000            |
| AWS         | 72,000            |


### 5. Most Optimal Skills to Learn
Combining insights from demand and salary data, this query aimed to pinpoint skills that are both in high demand and have high salaries, offering a strategic focus for skill development.
```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills AS skills,
        COUNT(job_postings_fact.job_id) AS demand_count

    FROM job_postings_fact
    INNER JOIN skills_job_dim on job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id

    WHERE 
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        
    GROUP BY   skills_dim.skill_id
),

average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(job_postings_fact.salary_year_avg),1) AS avg_salary

    FROM job_postings_fact
    INNER JOIN skills_job_dim on job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id

    WHERE 
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        
    GROUP BY skills_job_dim.skill_id
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary

FROM skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id

WHERE demand_count > 10

ORDER BY 
    demand_count DESC,
    avg_salary DESC
LIMIT 10;
```

| Skill ID | Skill       | Demand Count | Avg Yearly Salary (USD) |
|--------:|-------------|--------------|-------------------|
| 0       | SQL         | 3,083        | 96,435.3          |
| 181     | Excel       | 2,143        | 86,418.9          |
| 1       | Python      | 1,840        | 101,511.8         |
| 182     | Tableau     | 1,659        | 97,978.1          |
| 5       | R           | 1,073        | 98,707.8          |
| 183     | Power BI    | 1,044        | 92,323.6          |
| 188     | Word        | 527          | 82,940.8          |
| 196     | PowerPoint  | 524          | 88,315.6          |
| 186     | SAS         | 500          | 93,707.4          |
| 7       | SAS         | 500          | 93,707.4          |


# Insights
1. **Top-Paying Data Analyst Jobs:** The highest-paying jobs for data analysts in the UAE offer a wide range of salaries, from $72,000 being the lowest to the highest at $111,175 per year. Salaries in the UAE are closer to the global average of $93,876, albeit much lower than the global top-10 data analyst jobs, which pay salaries ranging from $285,000 to $650,000 a year.

2. **Skills for Top-Paying Jobs:** High-paying data analyst jobs require advanced proficiency in SQL, suggesting it’s a critical skill for earning a top salary. It is also interesting to note that PowerPoint showed up as a desired skill in the UAE, although it was demanded by the lowest-paying data analyst job.

3. **Most In-Demand Skills:** SQL is also the most demanded skill in the data analyst job market, followed by Excel and Python. The insights for demanded data analysis skills in the UAE mirror those for the global job market.

4. **Skills with Higher Salaries:** Specialized skills, such as SVN and Solidity, are associated with the highest average salaries globally, indicating a premium on niche expertise. The market in the UAE tells a different story, where Looker and R are the highest-paying skills. This indicates that employers in the UAE are looking for analysts who can not only analyze data programmatically (e.g., with R) but also translate insights into interactive dashboards using modern BI platforms like Looker. Additionally, UAE data analyst roles are less engineering-heavy relative to global markets like the US or Europe.

5. **Optimal Skills for Job Market Value:** SQL leads in demand and offers for a high average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value in the UAE and on a global scale.

# What I Learned
Through this project, I learned to leverage SQL's powerful data manipulation capabilities to derive meaningful insights from complex datasets:

- **Complex Query Construction:** Learning to build advanced SQL queries that combine multiple tables and employ functions like ```WITH``` clauses for temporary tables.
- **Data Aggregation:** Utilizing ```GROUP BY``` and aggregate functions like ```COUNT()``` and ```AVG()``` to summarize data effectively.
- **Analytical Thinking:** Developing the ability to translate real-world questions into actionable SQL queries that generate insightful answers.

# Conclusion
This project enhanced my SQL skills and provided valuable insights into the data analyst job market. The findings from the analysis serve as a guide to prioritizing skill development and job search efforts. Aspiring data analysts can better position themselves in a competitive job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of data analytics.
