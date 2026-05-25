# Introduction
This project analyzes remote Data Analyst job postings from 2023 to uncover:

The highest-paying roles

The most in-demand technical skills

The optimal skills for career growth (high demand + high salary)

# Background
The data field evolves rapidly. As an aspiring Data Analyst, I wanted to move beyond generic advice and answer specific questions:

Which companies pay top dollar for remote Data Analysts?

What skills drive the highest salaries?

Which skills offer the best "ROI" for learning (balance of demand and pay)?

## Tools I Used
SQL (PostgreSQL): Core analysis and data manipulation

VS Code: Query writing and testing

Git & GitHub: Version control and project sharing

# The Analysis
## 1. Top Paying Jobs
*Identifying the 10 highest-paying remote Data Analyst roles.*

```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_schedule_type,
    name as company_name,
    salary_year_avg,
    job_posted_date
FROM job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC
LIMIT 10;
```
Key Insights:

Top salary reaches $650,000 (Mantys)

6 of 10 top-paying jobs are at traditional companies (not tech giants)

Most require Senior or Director titles

## 2. Top-Paying Skills
*Skills associated with the highest-paying jobs (from the top 10 roles).*

```sql
WITH top_paying_skills AS (
    SELECT job_id, job_title_short, name as company_name, salary_year_avg
    FROM job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE job_title_short = 'Data Analyst' AND job_location = 'Anywhere' 
      AND salary_year_avg IS NOT NULL
    ORDER BY salary_year_avg DESC LIMIT 10
)
SELECT top_paying_skills.*, skills
FROM top_paying_skills
INNER JOIN skills_job_dim ON top_paying_skills.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC;
```
Key Insights:

SQL and Python appear in 80% of top-paying jobs

Cloud skills (AWS, Azure) command premium pay

Niche tools like PySpark and Hadoop appear frequently

## 3. Top Demanded Skills
Most frequently requested skills across ALL Data Analyst job postings.

```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) as demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst'
GROUP BY skills
ORDER BY demand_count DESC
LIMIT 5;
```
Skill	Demand Count
SQL	92,643
Excel	67,031
Python	57,326
Tableau	46,554
Power BI	39,468
Key Insight: SQL remains the undisputed king of data analysis.

## 4. Top Paying Skills
Skills that yield the highest average salary (any demand level).

```sql
SELECT 
    skills,
    ROUND(AVG(salary_year_avg), 0) as avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst' AND salary_year_avg IS NOT NULL
GROUP BY skills
ORDER BY avg_salary DESC
LIMIT 25;
```
Top 3 by Average Salary:

PySpark – $208,000

Bitbucket – $189,000

Couchbase – $160,000

Key Insight: Niche/big data skills pay significantly more than common ones.

## 5. Optimal Skills (Demand + Salary)
Skills with >10 job postings AND high average salary.

```sql
WITH skills_demand AS (
    SELECT skills_dim.skill_id, skills_dim.skills, COUNT(skills_job_dim.job_id) as demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE job_title_short = 'Data Analyst' AND salary_year_avg IS NOT NULL 
      AND job_work_from_home = TRUE
    GROUP BY skills_dim.skill_id
), average_salary AS (
    SELECT skills_job_dim.skill_id, ROUND(AVG(salary_year_avg), 0) as avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE job_title_short = 'Data Analyst' AND salary_year_avg IS NOT NULL 
      AND job_work_from_home = TRUE
    GROUP BY skills_job_dim.skill_id
)
SELECT skills_demand.skill_id, skills_demand.skills, demand_count, avg_salary
FROM skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE demand_count > 10
ORDER BY demand_count DESC, avg_salary DESC
LIMIT 25;
```
Optimal Skills (High Demand + High Pay):

Skill	Demand Count	Avg Salary
Python	148	$114,000
Tableau	140	$101,000
SQL	137	$108,000
R	91	$100,000
AWS	34	$135,000
Key Insight: Python + SQL + Tableau form the "golden trio" – learn these first.

# What I Learned
SQL is non-negotiable – appears in #1 demand and #2 optimal skills

Cloud skills pay off – AWS, Azure, GCP average $130k+ with solid demand

Specialization = higher salary – niche tools (PySpark, Hadoop) pay more but have lower demand

Remote jobs pay competitively – top salaries exceed many in-office roles


# Conclusions

If you're a Data Analyst looking to maximize career potential:

Master SQL first – it's the most practical ROI

Add Python and Tableau – high demand + high pay combination

Learn one cloud platform (AWS recommended) for senior roles

Don't ignore Excel – still #2 in demand after SQL