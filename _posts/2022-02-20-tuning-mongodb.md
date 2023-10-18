---
layout: post
title: "Optimize Slow Queries in MongoDB with Indexing"
author: pratyush
categories: [ Tuning, Tutorial ]
image: assets/images/tuning-mongodb.webp
description: "Optimize Slow Queries in MongoDB with Indexing"
comments: true
---

MongoDB is a popular NoSQL database known for its flexibility and scalability. However, as your data grows, you might encounter slow queries that can negatively impact your application's performance. One of the most effective ways to address this issue is by optimizing your queries through indexing.

**Identifying the Slow Query**
```
db.products.find({ category: "Electronics", price: { $gte: 500 } }).sort({ rating: -1 }).limit(10)
```

This query retrieves the top 10 highest-rated electronic products with a price greater than or equal to $500. However, as your dataset grows, executing this query might become slow and inefficient.

**Step 1**: Analyze Query Performance
The first step in optimizing a slow query is to analyze its performance. MongoDB provides a powerful tool called the `explain()` method, which helps you understand how MongoDB is executing your query. Run the slow query with `explain()` as follows
```
db.products.find({ category: "Electronics", price: { $gte: 500 } }).sort({ rating: -1 }).limit(10).explain("executionStats")
```

The output will provide detailed information about how MongoDB executed the query, including the execution time, the number of documents scanned, and more. Look for high values in fields like "executionTimeMillis" and "totalDocsExamined", as these are indicators of query performance issues.

**Step 2**: Create Indexes

Now that you've identified the slow query, it's time to optimize it with indexing. In MongoDB, indexes can significantly improve query performance by allowing the database to quickly locate the documents that match your query criteria.
```
db.products.createIndex({ category: 1, price: 1, rating: -1 })
```

This compound index includes the fields used in our query: `category`, `price` and `rating`. The order of fields in the index matters; it should match the query's filter criteria and sort order.

**Step 3**: Re-run the Query

Once the index is created, re-run the slow query. You should notice a significant improvement in query performance. Use the `explain()` method again to confirm the optimization
```
db.products.find({ category: "Electronics", price: { $gte: 500 } }).sort({ rating: -1 }).limit(10).explain("executionStats")
```

Results: Before and After Optimization

Before Indexing
* Execution time: 300 milliseconds
* Total documents examined: 100,000
* ExecutionTimeMillis: 300
* ExecutionSuccess: false

After Indexing
* Execution time: 5 milliseconds
* Total documents examined: 50
* ExecutionTimeMillis: 5
* ExecutionSuccess: true
