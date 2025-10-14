# Global-AI-Job-Market-Analysis
## Overview
This project involves a comprehensive analysis of the global AI job market from October 2024 to April 2025 using SQL and visualization tools. The goal is to extract valuable insights and answer key questions across four main areas: Hiring Market Overview, Salary Differences, Talent Profile, and Hiring Dynamics. The following README provides a detailed account of the project's objectives, analysis framework, business questions and solution code, dashboard visualizations, and conclusions.
## Objectives
- Identify where AI jobs are primarily located, what types of employment and work arrangements exist, and the approximate salary levels.
- Examine which structural factors mainly determine salary differences across positions.
- Explore the types of talent required for different job categories.
- Analyze which AI job sectors have the highest demand and how these trends evolve over time.
## Dataset
The data for this project is sourced from the Kaggle dataset: Global AI Job Market Trends & Salary Insights 2025  

**Brief Description:** The AI & Data Job Salaries and Skills Dataset 2024-2025 is synthetically prepared and provides a detailed, global overview of job listings in artificial intelligence, data science, machine learning, and related fields. This dataset aggregates job postings from October 2024 to July 2025, capturing a wide spectrum of roles, required skills, compensation, and organizational attributes across multiple countries and industries.  

**Dataset Link:** https://www.kaggle.com/datasets/pratyushpuri/global-ai-job-market-trend-2025
## Analysis Framework
### Module I. Market Overview: Global AI Job Landscape
- Global average salary for AI positions
- Distribution of AI jobs by job title
- Distribution of job postings by country (company_location)
- Distribution of AI jobs across industries
- Distribution of job types (full-time, part-time, contract, freelance)
- Distribution of high-benefits jobs (benefits_score ≥ 8)
### Module II. Salary Differences: Key Factors Affecting Salary
- Salary differences by country
- Salary differences by job type (full-time, part-time, contract, freelance)
- Salary differences across industries
- Salary differences by years of experience
- Salary differences between remote and on-site positions
- Salary differences by education level
### Module III. Talent Profile: Job and Education Match
- How job types correspond to education requirements
- Education requirements by job title
- Education level distribution across industries
- Proportion of jobs by experience level
### Module IV. Market Dynamics: Demand and Opportunities
- Top 10 most in-demand skills
- Top 10 most popular job positions
- Top 10 companies by number of job postings
- Monthly trend and cumulative count of AI job postings
- Recruitment cycle (average days a job remains open)
## Schema
```SQL
CREATE TABLE job
(
    job_id VARCHAR(8) 
	job_title VARCHAR(30) 
	salary_usd INT 
	salary_currency CHAR(3) 
	experience_level CHAR(2) 
	employment_type CHAR(2) 
	company_location VARCHAR(25) 
	company_size CHAR(1) 
	employee_residence VARCHAR(25) 
	remote_ratio TINYINT 
	required_skills VARCHAR(100) 
	education_required VARCHAR(50) 
	years_experience TINYINT  
	industry VARCHAR(25) 
	posting_date DATE 
	application_deadline DATE 
	job_description_length INT 
	benefits_score DECIMAL(3,1) 
	company_name VARCHAR(35)
);
```
## Solutions
### 1.Global average salary for AI positions
```sql
SELECT AVG(salary_usd) AS Global_average
FROM job;
```
### 2.Distribution of AI jobs by job title
```sql
SELECT 
    job_title,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / (SELECT COUNT(*) FROM job),
        2
    ) AS global_percentage
FROM job
GROUP BY job_title
ORDER BY global_percentage DESC;
```
### 3.Distribution of job postings by country (company_location)
```sql
SELECT 
    company_location AS country,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / (SELECT COUNT(*) FROM job),
        2
    ) AS global_percentage
FROM job
GROUP BY company_location
ORDER BY global_percentage DESC;
```
### 4.Distribution of AI jobs across industries
```sql
SELECT 
	industry,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / (SELECT COUNT(*) FROM job),
        2
    ) AS global_percentage
FROM job
GROUP BY industry
ORDER BY global_percentage DESC;
```
### 5.Distribution of job types (full-time, part-time, contract, freelance)
```sql
SELECT 
	employment_type,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / (SELECT COUNT(*) FROM job),
        2
    ) AS global_percentage
FROM job
GROUP BY employment_type
ORDER BY global_percentage DESC;
```
### 6.Distribution of high-benefits jobs (benefits_score ≥ 8)
```sql
SELECT 
    company_location,
    COUNT(*) AS total_jobs,
    SUM(CASE WHEN benefits_score >= 8 THEN 1 ELSE 0 END) AS high_benefit_jobs,
    ROUND(
        SUM(CASE WHEN benefits_score >= 8 THEN 1 ELSE 0 END) * 100.0 / COUNT(*),
        2
    ) AS high_benefit_ratio
FROM job
GROUP BY company_location
ORDER BY high_benefit_ratio DESC;
```
### 7.Salary differences by country
```sql
WITH country_avg AS (
    SELECT
        company_location AS country,
        ROUND(AVG(salary_usd), 2) AS avg_salary
    FROM job
    WHERE salary_usd IS NOT NULL
    GROUP BY company_location
)

SELECT
    country,
    avg_salary,
    ROUND(avg_salary - (SELECT ROUND(AVG(salary_usd),2) FROM job WHERE salary_usd IS NOT NULL), 2) AS diff_from_global_avg,
    DENSE_RANK() OVER (ORDER BY avg_salary DESC) AS salary_rank
FROM country_avg
ORDER BY salary_rank;
```
### 8.Salary differences by job type (full-time, part-time, contract, freelance)
```sql
SELECT 
	employment_type,
    AVG(salary_usd)
FROM job
GROUP BY employment_type
ORDER BY AVG(salary_usd) DESC;
```
### 9.Salary differences across industries
```sql
SELECT 
	industry,
    AVG(salary_usd)
FROM job
GROUP BY industry
ORDER BY AVG(salary_usd) DESC;
```
### 10.Salary differences by years of experience
```sql
SELECT 
    experience_level,
	AVG(salary_usd)
FROM job
GROUP BY experience_level;
```
### 11.Salary differences between remote and on-site positions
```sql
SELECT 
    remote_ratio,
    AVG(salary_usd) AS avg_salary
FROM job
GROUP BY remote_ratio;
```
### 12.Salary differences by education level
```sql
SELECT
	education_required,
    AVG(salary_usd) AS avg_salary_by_education
FROM job
GROUP BY education_required;
```
### 13.How job types correspond to education requirements
```sql
SELECT
    employment_type,
    education_required,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY employment_type),
        2
    ) AS education_percentage
FROM job
WHERE education_required IS NOT NULL
GROUP BY employment_type, education_required
ORDER BY employment_type, education_percentage DESC;
```
### 14.Education requirements by job title
```sql
SELECT
    job_title,
    education_required,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY job_title),
        2
    ) AS percentage_by_job
FROM job
WHERE education_required IS NOT NULL
GROUP BY job_title, education_required
ORDER BY job_title, percentage_by_job DESC;
```
### 15.Education level distribution across industries
```sql
SELECT
    industry,
    education_required,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY industry),
        2
    ) AS percentage_by_industry
FROM job
WHERE education_required IS NOT NULL
GROUP BY industry, education_required
ORDER BY industry, percentage_by_industry DESC;
```
### 16.Proportion of jobs by experience level
```sql
SELECT
    experience_level,
    COUNT(*) AS job_count,
    ROUND(
        COUNT(*) * 100.0 / (SELECT COUNT(*) FROM job),
        2
    ) AS percentage_of_total
FROM job
WHERE experience_level IS NOT NULL
GROUP BY experience_level
ORDER BY job_count DESC;
```
### 17.Top 10 most in-demand skills
```sql
WITH skill_exploded AS (
    SELECT 
        TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(required_skills, ',', n.n), ',', -1)) AS skill
    FROM 
        job
    JOIN (
        SELECT 1 AS n UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5
        UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9 UNION SELECT 10
    ) AS n
    ON CHAR_LENGTH(required_skills)
       - CHAR_LENGTH(REPLACE(required_skills, ',', '')) >= n.n - 1
)
SELECT 
    skill,
    COUNT(*) AS frequency
FROM skill_exploded
WHERE skill <> ''
GROUP BY skill
ORDER BY frequency DESC
LIMIT 10;
```
### 18.Top 10 most popular job positions
```sql
WITH job_count AS (
    SELECT
        job_title,
        COUNT(*) AS frequency
    FROM job
    WHERE job_title IS NOT NULL
    GROUP BY job_title
)
SELECT
    job_title,
    frequency,
    DENSE_RANK() OVER (ORDER BY frequency DESC) AS rank_job
FROM job_count
ORDER BY rank_job
LIMIT 10;
```
### 19.Top 10 companies by number of job postings
```sql
SELECT
    company_name,
    COUNT(*) AS job_count,
    DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS rank_count
FROM job
GROUP BY company_name
ORDER BY rank_count
LIMIT 10;
```
### 20.Monthly trend and cumulative count of AI job postings
```sql
SELECT
    DATE_FORMAT(posting_date, '%Y-%m') AS posting_by_month,
    COUNT(*) AS monthly_jobs,
    SUM(COUNT(*)) OVER (ORDER BY DATE_FORMAT(posting_date, '%Y-%m')) AS cumulative_jobs
FROM job
GROUP BY posting_by_month
ORDER BY posting_by_month;
```
### 21.Recruitment cycle (average days a job remains open)
```sql
SELECT
	job_title,
    ROUND(AVG(DATEDIFF(application_deadline, posting_date)), 0) AS avg_days_to_deadline
FROM job
WHERE application_deadline IS NOT NULL
GROUP BY job_title
ORDER BY avg_days_to_deadline DESC;
```
## Dashboard Visualizations
## Findings and Conclusion
