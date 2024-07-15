INFLUENCER MARKTING ANALYTICS QUERY
This SQL query is designed to create a comprehensive view of social media campaign performance, aggregating data from various sources and performing complex calculations. The view, named adhoc.nexus_mart_view_s, incorporates several advanced SQL techniques such as the use of common table expressions (CTEs), conditional calculations, and string pattern matching with regular expressions.

Key Features
Total Media Value (TMV) Calculation: Calculates the TMV for different social media networks using a custom formula.
Regular Expressions: Uses regex to identify mall locations from post captions.
UNION ALL: Combines live and non-live member data into a single dataset.
Creator Status and Tier Calculation: Determines the status and tier of content creators based on their activity and follower count.
Steps Explained
Calculate Live Members: Identifies and categorizes live members from the bi.mention and bi.member tables.
Daily Aggregates: Aggregates daily post performance metrics for live members, including impressions, likes, comments, etc. It also calculates TMV based on network-specific formulas.
Approved Members: Retrieves a list of approved members and excludes live members from this list to identify non-live approved members.
Content Reviews: Filters content reviews for non-live members to identify their current status.
Combine Data: Merges data from live members and other creator statuses into a single dataset using UNION ALL.
Social Account Data: Enriches the combined data with social account details.
Campaign and Mall Location: Adds campaign names and mall locations based on regex matches in post captions.
