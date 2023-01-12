# 系统设计需要考虑的点

针对系统设计的题目，通常需要考虑：
1、可行的解；
2、根据具体场景，系统偏向特性，对比、权衡多个解；
3、先分析问题，设计的关键点，可以采取哪些手段解决，最后才是技术选型；
4、从小到大，一步步涉及，不要不计成本的天马行空；

# 思路

1、 确定场景

- 具体需要什么功能
- 具体的性能要求(QPS)
  - 万级别QPS就需要考虑集群，涉及到集群就要考虑单点故障、高可用、
- 找准核心功能点：如果是设计稍微大一点的系统，抓重点；