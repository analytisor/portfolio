# **INFLUENCER MARKETING QUERY**

This SQL query is designed to create a comprehensive view of social media campaign performance, aggregating data from various sources and performing complex calculations. The view, named **`adhoc.nexus_view_s`**, incorporates several advanced SQL techniques such as the use of common table expressions (CTEs), conditional calculations, string pattern matching with regular expressions, and the **`UNION`** function to combine different segments of data.

## **<u>Key Features</u>**

- **<u>Total Media Value (TMV) Calculation</u>**: Calculates the TMV for different social media networks using a custom formula.
- **<u>Regular Expressions</u>**: Uses regex to identify mall locations from post captions.
- **<u>UNION ALL</u>**: Combines live and non-live member data into a single dataset.
- **<u>Creator Status and Tier Calculation</u>**: Determines the status and tier of content creators based on their activity and follower count.

## **<u>Steps Explained</u>**

1. **<u>Calculate Live Members</u>**: Identifies and categorizes live members from the **`bi.mention`** and **`bi.member`** tables.
2. **<u>Daily Aggregates</u>**: Aggregates daily post performance metrics for live members, including impressions, likes, comments, etc. It also calculates TMV based on network-specific formulas.
3. **<u>Approved Members</u>**: Retrieves a list of approved members and excludes live members from this list to identify non-live approved members.
4. **<u>Content Reviews</u>**: Filters content reviews for non-live members to identify their current status.
5. **<u>Combine Data</u>**: Merges data from live members and other creator statuses into a single dataset using **`UNION ALL`**.
6. **<u>Social Account Data</u>**: Enriches the combined data with social account details.
7. **<u>Campaign and Mall Location</u>**: Adds campaign names and mall locations based on regex matches in post captions.
