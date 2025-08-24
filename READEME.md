# Pulse Analytics

This is requirements of system design for Data Pipelines

### In a food delivery platform, management would like dashboards summarising key user metrics such as conversion funnels from marketing campaigns, orders, and key logistics metrics such as delivery time end to end

- From separate application systems: orders, logistics, promotions, payments
- Third party vendor data such as customer support logs, Google Analytics, photos

### Performance targets

- Throughput: data volume of x TB raw data daily, to be completed in 6 hours from start to finish
- Latency
- Error rate
- Alerts and pipeline monitoring
- Budget of $xk monthly

### Issue:

- Decentralised analytics: everyone pulls their own data and does their own transformation and
- visualisation
- No central knowledge base / definitions of key metrics
