1.) what is observability?

Ans.) if we have observability setup we can get the information of internal state system (app+infra+network)
      we can get the information of disk utilization of particular pod
      we can get the information cpu utilization of yhe node
      memory utilization
      we can get http requests success or failure
2.) why is observability?

Ans.) By using observability we can find the failures and also helps to fix it 

3.) what is metrics?

Ans.) Metrics basically gives historical data of event
      event means: cpu,memory,disk,http requests

4.) what is logs?

Ans.) it will helps to understand application better
      different logs: info log,debug log, error logs, trace log

5.) what is traces?

Ans.) by using traces we can troubleshoot, fixing theissues

---------------------------Promethues Architecture:------------------------------------------------------------------------


 +------------------+        +------------------+
 |  Exporters       |        |   Push Gateway   |
 | (Node, App, etc) |        | (optional)       |
 +--------+---------+        +--------+---------+
          |                           |
          v                           v
   +--------------------------------------+
   |         Prometheus Server            |
   |  - Time Series Database              |
   |  - Data Retrieval Engine (scraping)  |
   |  - TSDB Storage                      |
   |  - Alert Manager Integration         |
   +----------------+---------------------+
                    |
                    v
         +------------------------+
         |    Alertmanager        |
         | (Email, Slack, etc.)   |
         +------------------------+
                    |
                    v
         +------------------------+
         |     Grafana (UI)       |
         | (Queries, Dashboards)  |
         +------------------------+


🔄 Scraping Workflow

  . Prometheus periodically scrapes endpoints (e.g., /metrics) exposed by exporters.

  . It stores the collected metrics in its TSDB (time series database).

  . Prometheus can trigger alerts based on alert rules.

  . Dashboards in Grafana pull data using PromQL to visualize metrics.

  pometheus scrapes metrics from node exporter(cpu, memory), kube state metrics(Api server), custom metrics(http request, login authentication) and stores them in timeseries database

**custom metrices:**
    Instrumentation: when developer setup metrics,logs, traces in the application then that is called as instrumentation
   
   **Metrics:** Total 4 types of metrics we have 
                1.) **counter:** eg: logging into the application or HTTp request (when user logged in the application count will be increase that comes under counter metric type) 
                2.) **gauge** eg: cpu utilization(In cpu utilization it will increase or decrease so we can guage metrics )
                3.) **histogram**In histogram will create buckets like(Anyone ask how many times my https request taking more than 5ms or how many times cpu utilization is more than 50 percent this scenario 
                                                                       will use this histogram metrics)
                 4.) **summary**:








**Grafana:**

-> it is open source tool for performing data analytics retrieving metrics that make of large amount of data and monitoring our apps using nice configurable dashboard.

-> so grafana can integrate with a wide range of data sources including graphite,prometheus influx TB elastic search mysql postgres and others.

-> so when connected to supported data sources it will provide web based chart graphs and alert

**promethus**

-> it is an open-source monitoring and alerting toolkit designed for reliability and scalability in model and dynamic environment.

-> which prometheus is excels at collecting and storing time series data allowing users to gain valuable insight into the performance and health of the application and infrastruture 

-> so its powerful query language and support for multi-dimensional data prometheus has become a popular choice for monitoring system with cloud native



