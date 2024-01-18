---

title: What is Skodjob performance hub?
menu_order: 1
post_status: publish
post_excerpt: Are you interested in performance testing/tuning your project? If the answer is yes check out Skodjob performance hub!
featured_image: _images/perf-intro.jpg
taxonomy:
    category:
        - performance

---

## What is our goal?
Performance is and has always been something cursed in some way. Everyone knows that it should be important
but nobody tries to do it. Well, we would like to **change this**!

Performance hub is the source of all tooling that we are working on. We hope that these tools will
give you enough confidence to start **performance tuning** your software. Well as we are mostly practical folks
lets move to what is currently included in [performance hub](https://github.com/skodjob/database-performance-hub).

### Database Manipulation Tool
Imagine the situation when you working on a product that supports multiple database engines and you
would like to perform some tuning. Well, that sucks! You have to create different loads because
most of the databases use different dialects. This is certainly not something that will encourage
you in this action. Believe us we have been in this situation. Gladly we have a solution for you!

[DMT (database-manipulation-tool)](https://github.com/skodjob/database-performance-hub/tree/main/database-manipulation-tool)
is a Quarkus application that allows you to specify data load in `JSON` and the tool will take care of the rest.
So far we have implemented `PostgreSQL, MongoDB, and MySQL` dialects and we aim for more!
Don't worry we didn't forget about the perf part. This tool is currently developed in two modes
performance and production. The performance one is stripped of all things that could influence
The results of the performance tests or tuning. Production one is more user-friendly, specifically the purpose of
integration tests.

### Load generator


### Automation