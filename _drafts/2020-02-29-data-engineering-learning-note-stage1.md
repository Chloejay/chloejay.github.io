I was trying to combine the pieces and learn some use case from some big FAANG and etc to get understand what's the typical 
data engineering project it covers, it varied. 

Like everything, learning wise is to organize the old knowledge/experience with new gains, mixed them together to build my own understanding among those, which is "mine" then, I call this process is building my knowledge framework. 

So before digging into, understand the frame and principle is the key, which makes me easy to enter, for I need to follow the rule. 
In the data engineering field, some rule to follow,
1. design for immutable data 
2. create data lineage 
3. gradual and optional data validation (QA) 
4. build the data catelog 
5. use the tool to manage the data version, critically important when building models 

The typical step come from data ingestion, to data modeling, data transformation and analytics, dashboarding and deep learning, etc. ETL is the core task that data engineer needs to deal with daily, the most high level of desgn data architecture is just ETL, fetch data, transform data, like do the aggregation and enrichment, load to the desintation for the busienss purpose. On the low level, it's complex enough which needs the knowledge of software engineering, system design, database development, for data is dirty and it comes from differnt places, be it log files, api, databases. For the modern data engineering, real time stremaing is popular, for it meets the business demands, so deal with large volume datasets, batch processing can't handle it in low latency. So new tool like Spark streaming or Kafka streaming is being used to do the data processing, such as aggregation, enrichment and standardization etc. 

immutable can make sure no matter how many times we process data, it will consitent, be idempotent, which is 1/4 elements of ACID. 

data lineage, the transformed data can be tracked at different stages, like the airflow we used to execute the task when we run the bashoperator or pythonoperator for each task. 