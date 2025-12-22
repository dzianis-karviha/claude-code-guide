# 10. Results

Subjectively, I'd estimate the performance gain at 30-40% in comparison with the period before the Claude Code introduction. So, it's definitely more than a week saved per month.

I decided to gather the statistics based on my git history for the last two-year period.

I understand that it cannot be used as the only valid metric for measuring performance. At the same time, I think that it has a correlation with productivity if we compare it for long periods for the same codebase, and with similar types of tasks done during these periods.

I've chosen the following periods to compare:

* May - August 2024
* August - October 2024
* October 2024 - January 2025
* January - March 2025
* March - June 2025
* June - August 2025
* October - December 2025

September 2025 is excluded because it was a period for different experiments with Claude Code, and it's not relevant for comparison.

All the statistic is aggregated by weeks. Weeks with fewer than 10 git commits in a week are excluded from the calculation (in order to exclude vacations, etc). Each of the periods above has 9-11 weeks, matching this requirement.

Generally, all these periods are ongoing development of the application. At the same time, they could have some features impacting the LOC statistic.

Here are summary comparison table for the statistics and some graphs. More detailed data is available via the links at the end of this section in interactive graph view.

|            Metric             | May-Aug'24  | Aug-Oct'24  | Oct-Jan'25  | Jan-Mar'25  | Mar-Jun'25  | Jun-Aug'25  | Oct-Dec'25  |
| :---------------------------: | :---------: | :---------: | :---------: | :---------: | :---------: | :---------: | :---------: |
|        Weeks included         |     11      |      9      |     11      |     10      |     11      |     11      |     11      |
|                               |             |             |             |             |             |             |             |
|     Avg Commits per Week      |     30      |     25      |     30      |     25      |     26      |     28      |     50      |
| Avg Prod Deployments per Week |     21      |     19      |     20      |     19      |     22      |     22      |     28      |
|                               |             |             |             |             |             |             |             |
|      Total NET LOC - Min      |    1,419    |     374     |     499     |     959     |    2,288    |    1,369    |    2,801    |
|      Total NET LOC - Max      |    3,413    |    3,306    |    3,815    |    3,870    |    7,062    |    4,297    |   12,143    |
|   ***Total NET LOC - Avg***   | ***2,590*** | ***1,681*** | ***2,045*** | ***2,481*** | ***3,975*** | ***2,506*** | ***5,947*** |
|                               |             |             |             |             |             |             |             |
|     Source NET LOC - Min      |     925     |     377     |     454     |     765     |    2,042    |    1,061    |     988     |
|     Source NET LOC - Max      |    2,871    |    3,094    |    3,696    |    3,132    |    5,349    |    3,025    |    6,799    |
|  ***Source NET LOC - Avg***   | ***2,125*** | ***1,480*** | ***1,701*** | ***2,016*** | ***3,267*** | ***1,822*** | ***3,473*** |
|                               |             |             |             |             |             |             |             |
|      Test NET LOC - Min       |     88      |    -232     |     45      |     159     |     129     |     132     |     296     |
|      Test NET LOC - Max       |     924     |     661     |     886     |     953     |    1,713    |    1,988    |    5,091    |
|   ***Test NET LOC - Avg***    |  ***434***  |  ***145***  |  ***344***  |  ***463***  |  ***707***  |  ***505***  | ***2,043*** |

The following metrics are used:

* Weeks Included - the number of weeks used for calculation
* Avg Commits per Week - the metric is not a direct reference for performance. Commits with Claude Code became more atomic (previously, I could combine several changes into a single commit to save time). That is also better from a CI/CD principles perspective and makes it easier to apply automated code review by a clear scope for each commit.
* Avg Prod Deployments per Week - this is an *approximate* number of production deployments gathered as the number of git tags created during this period. Git tag is used as the version of the production build. Not all of them are actually deployed to production. At the same time, several components are deployed from a single git repo and only 1 tag is used for them
* Source NET LOC - * - metric calculated for source files only as NumberOfAdded minus NumberOfDeleted
* Test NET LOC - * - metrics calculated for test files only
* Total NET LOC - both, source files + test files

And here is the graph that compares NET LOC statistic of October - December 2025 with other periods

![img.png](sections/performance.png)

For more detailed metrics, see the interactive charts:

[Weekly Statistic](sections/./weekly_stats.html) | [Period Statistic](sections/./period_stats.html)
