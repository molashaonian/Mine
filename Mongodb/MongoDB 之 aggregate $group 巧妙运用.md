有这样一组数据：
```
{
    "campaign_id": "A",
    "campaign_name": "A",
    "subscriber_id": "123"
},
{
    "campaign_id": "A",
    "campaign_name": "A",
    "subscriber_id": "123"
},
{
    "campaign_id": "A",
    "campaign_name": "A",
    "subscriber_id": "456"
}
```
按照  campaign_id 与 campaign_name  分组，并查询出每个分组下的记录条数 及  subscriber_id  不同记录的个数

关系型数据库SQL示例：
```
select campaign_id,campaign_name,count(subscriber_id),count(distinct subscriber_id)
group by campaign_id,campaign_name from campaigns;
```
在MongoDB下就存在两种组合：1） campaign_id, campaign_name, subscriber_id  三个相同的分为一组，
						
						   2） campaign_id, campaign_name  两个相同，subscriber_id 不同分为一组，

最后通过这两种分组查询出按照  campaign_id 与 campaign_name  分组，subscriber_id 不同记录的个数

MongoDB示例：
```
db.campaigns.aggregate([
    { "$match": { "subscriber_id": { "$ne": null }}},

    // Count all occurrences
    { "$group": {
        "_id": {
            "campaign_id": "$campaign_id",
            "campaign_name": "$campaign_name",
            "subscriber_id": "$subscriber_id"
        },
        "count": { "$sum": 1 }
    }},

    // Sum all occurrences and count distinct
    { "$group": {
        "_id": {
            "campaign_id": "$_id.campaign_id",
            "campaign_name": "$_id.campaign_name"
        },
        "totalCount": { "$sum": "$count" },
        "distinctCount": { "$sum": 1 }
    }}
])
```

文档结果：

第一个 group：
```
{ 
    "_id" : { 
        "campaign_id" : "A", 
        "campaign_name" : "A", 
        "subscriber_id" : "456"
    }, 
    "count" : 1 
}
{ 
    "_id" : { 
        "campaign_id" : "A", 
        "campaign_name" : "A", 
        "subscriber_id" : "123"
    }, 
    "count" : 2
}
```
第二个 group：
```
{ 
    "_id" : { 
        "campaign_id" : "A", 
        "campaign_name" : "A"
    },
    "totalCount" : 3,
    "distinctCount" : 2
}
```
至此，我们已经查询出一共有 3 条记录，subscriber_id 有两种不同的值

reference： [Mongodb中Aggregation特性](http://shift-alt-ctrl.iteye.com/blog/2259216)
