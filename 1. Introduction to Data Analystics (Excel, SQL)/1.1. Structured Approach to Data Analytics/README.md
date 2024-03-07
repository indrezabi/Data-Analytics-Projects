# Structured Approach to Data Analytics
## Task
It is the very first task of the entire data analytics course. The objective was to choose the topic from our own everyday life and conduct a small analysis to become acquainted with the six basic steps of data analysis.

## Solution
I chose to analyze my sleep using the data extracted from my smartwatch. Considering the complexity of the data, I decided to use BigQuery for the very first time and cleaned the data using the following simple code:
```
SELECT 
    bedtime,
    asleep,
    quality,
    deep,
    sleepBPM

FROM 
    `m1s1p3-sleepdata.SleepData.SleepData2023`

WHERE 
    fromdate LIKE '%Mar%'
    AND todate LIKE '%Mar%'
    ```
After that, I completed all six steps of analysis and presented my project and results in this presentation (which can be found [here](https://docs.google.com/presentation/d/13KFu8w1_BHDRxafzZmCM_FWYnRGPOda2BQWAeRVYIAI/edit?usp=sharing)).

