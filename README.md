# 研究動機
深入探索數據領域的就業市場！本專案聚焦於數據分析師職位，探討高薪職缺、熱門技能，以及數據分析領域中高需求與高薪資交匯的關鍵點。

# 研究背景
本專案源自於對數據分析師就業市場的深入探索，旨在精確識別高薪且需求量大的技能，幫助他人更高效地尋找理想職位。

資料來自於Luke的 [SQL課程](https://lukebarousse.com/sql)。 其中包含了職位名稱、薪資、地點及關鍵技能等豐富的見解。

### 我希望透過 SQL 解答的問題：

1. 哪些是薪資最高的數據分析師職位？

2. 這些高薪職位需要哪些技能？

3. 數據分析師最需要哪些技能？

4. 哪些技能與較高薪資相關？

5. 最值得學習的技能是什麼？
# 分析結果
每個針對此專案的查詢目的在調查數據分析師職場的特定面向。以下是我處理每個問題的方法：

### 1. 薪資最高的數據分析師職位
為了識別薪資最高的職位，我根據年均薪資和地點篩選了數據分析師職位，並設定為遠端工作。

```sql
SELECT	
	job_id,
	job_title,
	job_location,
	job_schedule_type,
	salary_year_avg,
	job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
```
以下是 2023 年薪資最高的數據分析師職位概況：

- **廣泛的薪資範圍**： 薪資最高的前 10 名數據分析師職位年薪從 184,000 美元到 650,000 美元不等，顯示該領域的薪資潛力非常大。

- **多種產業的需求**： 包括 SmartAsset、Meta 和 AT&T 等公司都提供高薪，顯示不同產業對數據分析師的廣泛需求。

- **職位種類多樣**： 從數據分析師到分析總監等職位種類繁多，反映了數據分析領域內不同角色和專業領域的多樣性。

![高薪職位](pictures/1_top_paying_roles.png)
*此為數據分析師薪資排名前 10 名的長條圖；這張圖是 ChatGPT 根據我的 SQL 查詢結果生成的。*

### 2. 高薪職位所需技能
為了了解高薪職位所需的技能，我將職位招聘信息與技能數據進行聯接，分析高薪職位所需的技能。

```sql
WITH top_paying_jobs AS(
    SELECT	
	job_id,
	job_title,
	salary_year_avg,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC
```
以下是 2023 年薪資最高的數據分析師職位中最需求的技能概況：

- **SQL** 出現 8 次為最多。

- **Python** 出現了 7 次。

- **Tableau** 也非常熱門，出現了 6 次。 其他技能如 **R**、**Snowflake**、**Pandas** 和 **Excel** 也顯示了不同程度的需求。

![高薪技能](pictures/2_top_paying_roles_skills.png)
*此為薪資最高的數據分析師職位中技能出現次數的長條圖；這張圖是 ChatGPT 根據我的 SQL 查詢結果生成的。*

### 3. 數據分析師需求技能

這個查詢幫助了解在職位招聘中需要的技能，特別是那些需求量高的。

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5
```
以下是 2023 年數據分析師需求最高的技能概況：

- **SQL** 和 **Excel** 仍然是基礎，強調了在數據處理和表格操作中對扎實基礎技能的需求。

- **程式語言** 和 **資料視覺化工具** 如 **Python**、**Tableau** 和 **Power BI** 等，顯示了數據故事呈現和決策輔助的重要性日益增加。

| Skills   | Demand Count |
|----------|--------------|
| SQL      | 7291         |
| Excel    | 4611         |
| Python   | 4330         |
| Tableau  | 3745         |
| Power BI | 2609         |

*此為數據分析師職位招聘中前 5項技能需求的表格*

### 4. 不同技能對應的薪資
通過探索不同技能相關的平均薪資，可以找出哪些技能是薪資最高的。
```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg),0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' 
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25
```
以下是數據分析師薪資最高技能的概況：

- **大數據與機器學習技能的高需求**： 擅長大數據技術（如 PySpark、Couchbase）、機器學習工具（如 DataRobot、Jupyter）以及 Python（如 Pandas、NumPy）的分析師擁有最高薪資，反映了行業對數據處理和預測建模能力的高度重視。

- **軟件開發與部署能力**： 了解開發和部署工具（如 GitLab、Kubernetes、Airflow）顯示了數據分析與工程之間的交集，對促進自動化和高效數據管道管理的技能有更高的需求。

- **雲端專業知識**： 熟悉雲端和數據工程工具（如 Elasticsearch、Databricks、GCP）突顯了雲端環境日益重要，這表明雲端專業能力顯著提高了數據分析領域的薪資潛力。

| Skills        | Average Salary ($) |
|---------------|-------------------:|
| pyspark       |            208,172 |
| bitbucket     |            189,155 |
| couchbase     |            160,515 |
| watson        |            160,515 |
| datarobot     |            155,486 |
| gitlab        |            154,500 |
| swift         |            153,750 |
| jupyter       |            152,777 |
| pandas        |            151,821 |
| elasticsearch |            145,000 |

*此為數據分析師平均薪資前十高的技能表格*

### 5. 最值得學習的技能
結合需求和薪資數據，這個查詢希望找出既有高需求又有高薪資的技能，為技能學習提供策略性重點。
```sql
SELECT 
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = True 
GROUP BY
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```
以下是2023年數據分析師最值得學習的技能詳細介紹：

- **高需求的程式語言**： Python和R在需求上脫穎而出，需求量分別為236和148。儘管需求高，Python的平均薪資約為$101,397，R為$100,499，這表明對這些語言的熟練度高度重視，但同時也非常普遍。

- **雲端工具和技術**： 例如Snowflake、Azure、AWS和BigQuery等專業技術的技能需求顯著，並且擁有相對較高的平均薪資，這顯示出雲端平台和大數據技術在數據分析中的日益重要性。

- **商業智慧與可視化工具**： Tableau 和 Looker 的需求量分別為 230 和 49，平均薪資約為 $99,288 和 $103,795，這顯示資料視覺化和商業智慧工具在從數據中提煉可操作洞察方面的重要性。

- **資料庫技術**： 針對傳統和NoSQL資料庫（如Oracle、SQL Server、NoSQL）的技能需求，平均薪資範圍從$97,786到$104,534，反映了對數據存儲、檢索和管理專業知識的持續需求。

| Skill ID | Skills     | Demand Count | Average Salary ($) |
|----------|------------|--------------|-------------------:|
| 8        | go         | 27           |            115,320 |
| 234      | confluence | 11           |            114,210 |
| 97       | hadoop     | 22           |            113,193 |
| 80       | snowflake  | 37           |            112,948 |
| 74       | azure      | 34           |            111,225 |
| 77       | bigquery   | 13           |            109,654 |
| 76       | aws        | 32           |            108,317 |
| 4        | java       | 17           |            106,906 |
| 194      | ssis       | 12           |            106,683 |
| 233      | jira       | 20           |            104,918 |
| 79       | oracle     | 37           |            104,534 |
| 185      | looker     | 49           |            103,795 |
| 2        | nosql      | 13           |            101,414 |
| 1        | python     | 236          |            101,397 |
| 5        | r          | 148          |            100,499 |
| 78       | redshift   | 16           |             99,936 |
| 187      | qlik       | 13           |             99,631 |
| 182      | tableau    | 230          |             99,288 |
| 197      | ssrs       | 14           |             99,171 |
| 92       | spark      | 13           |             99,077 |

*此為依平均薪資排序的數據分析師必學技能表格*


# 結論

### 從分析結果中發現的一些趨勢

1. **高薪數據分析師工作**： 允許遠程工作的數據分析師高薪職位提供了廣泛的薪資範圍，最高可達65萬美元！

2. **高薪職位所需技能**： 高薪數據分析師工作需要在SQL方面的高級能力，這表明SQL是獲得高薪的關鍵技能。

3. **需求最高的技能**： SQL也是數據分析師市場上需求最高的技能，因此對求職者來說是必須掌握的技能。

4. **薪資較高的技能**： 專業技能，如SVN和Solidity，與最高的平均薪資相關，顯示出小眾專業知識的價值。

5. **最適合學習的技能以提高市場價值**： SQL在需求和薪資上領先，是數據分析師學習以最大化市場價值的最適技能之一。

### 結語

這個專案不僅提升了我的 SQL 技能，還讓我對數據分析師職場市場有了更深入的認識。分析結果為技能培養和求職策略提供了明確的方向，特別是如何透過專注於高需求、高薪資的技能，在競爭激烈的市場中脫穎而出。這項研究進一步強調了數據分析領域中，持續學習與適應新興趨勢的重要性。
