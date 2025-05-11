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


