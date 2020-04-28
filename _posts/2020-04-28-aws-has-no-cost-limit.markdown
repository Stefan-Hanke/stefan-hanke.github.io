---
layout: post
title:  "AWS has no cost limit"
date:   2020-04-28
categories: iac
---

When working with AWS, I want to be sure that costs stays reasonable.

Unfortunately, there is no easy way to actual *limit* costs. You just can create [budgets](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/budgets-managing-costs.html) for which you can be notified. I've setup the budget to be a reasonably low sum, plus two alerts that write an E-Mail to me at 50% at 80% spent budget.

# A homegrown solution

While doing a 5-minute search, I came across [this reddit post](https://www.reddit.com/r/aws/comments/3bou1p/is_there_no_way_to_limit_costs/). One answer is:

> One option is to use the notification to trigger a stop for whatever service/resource. I'm sure you could write up a quick script in conjunction with the AWS CLI.

This looks easy enough that somebody may have already implemented this solution and blogged about it. I'm sure I'll do something like that once cost get beyond my personal feel-good-level.