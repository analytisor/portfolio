DROP VIEW IF EXISTS adhoc.nexus_stage_view_s;

CREATE OR REPLACE VIEW adhoc.nexus_stage_view_s AS
-- Step 1: Calculate live creators
WITH live_creators AS (
    SELECT
        mn.member_id,
        mn.program_id,
        mn.post_id,
        mn.client_id::text,
        mn.id AS mention_id,
        mn.community_id,
        mn.cost,
        mn.project_id,
        'LIVE' AS creator_status,
        m.name AS member_name
    FROM bi.mention mn
    JOIN bi.member m ON mn.member_id = m.id
    WHERE mn.client_id = 'wpNVzQZ7FoyqUaqb75WP1XBgUTEyLzvZ'
),

-- Step 2: Get all approved members and join with content review to create different statuses
approved_members AS (
    SELECT
        m.id AS member_id,
        m.name AS member_name
    FROM bi.member m
    JOIN bi.program_membership pm ON m.id = pm."memberId"
    WHERE m."clientId" = 'wpNVzQZ7FoyqUaqb75WP1XBgUTEyLzvZ'
    AND pm.status = 'approved'
),

content_status AS (
    SELECT
        cr.member_id,
        cr.post_id,
        cr.id AS content_review_id,
        cr.campaign_id,
        cr.state AS content_review_state,
        cr.project_id,
        cr.datetime_created,
        cr.datetime_modified,
        cr.date_last_reject,
        cr.time_to_mark_complete,
        am.member_name,
        NULL::text AS client_id,
        NULL::bigint AS mention_id,
        NULL::bigint AS community_id,
        NULL::bigint AS cost,
        NULL::bigint AS program_id,
        CASE
            WHEN cr.state = 'PLACEHOLDER' THEN 'LOCKED'
            WHEN cr.state = 'ACCEPTED' THEN 'APPROVED'
            WHEN cr.state IN ('REJECTED', 'AMENDED') THEN 'CONTENT REVIEW'
            WHEN cr.state = 'MARKED_COMPLETE' THEN 'UGC COMPLETE'
            WHEN cr.state = 'COMPLETED' THEN 'DROP AND PAY'
            WHEN cr.post_id IS NOT NULL THEN 'NOT RECOGNIZED'
            ELSE 'NOT RECOGNIZED'
        END AS creator_status
    FROM bi.content_review cr
    JOIN approved_members am ON cr.member_id = am.member_id
    WHERE cr.member_id NOT IN (SELECT member_id FROM live_creators)
),

-- Step 3: Combine live creators with the rest of the content status
all_creators AS (
    SELECT 
        lc.member_id,
        lc.program_id,
        lc.post_id,
        lc.client_id,
        lc.mention_id,
        lc.community_id,
        lc.cost,
        lc.project_id,
        lc.creator_status,
        lc.member_name
    FROM live_creators lc
    UNION ALL
    SELECT 
        cs.member_id,
        cs.program_id::bigint,
        cs.post_id,
        cs.client_id::text,
        cs.mention_id::bigint,
        cs.community_id::bigint,
        cs.cost::bigint,
        cs.project_id::bigint,
        cs.creator_status,
        cs.member_name
    FROM content_status cs
),

-- Step 4: Join with campaign data to get campaign name
campaign_data AS (
    SELECT
        ac.*,
        p.title AS campaign_name
    FROM all_creators ac
    LEFT JOIN bi.program p ON ac.program_id = p.id
),

-- Step 5: Join with agreement data to get agreement ID and date agreement agreed upon
agreement_data AS (
    SELECT
        cd.member_id,
        cd.program_id,
        cd.project_id,
        cd.post_id,
        cd.client_id,
        cd.mention_id,
        cd.community_id,
        cd.cost,
        cd.creator_status,
        cd.member_name,
        cd.campaign_name,
        a.id AS agreement_id,
        a.date_agreement_agreed_upon
    FROM campaign_data cd
    LEFT JOIN bi.agreement a ON cd.member_id = a.member_id AND cd.project_id = a.project_id
),

-- Step 6: Join with agreement_iteration data to get id and price
agreement_iteration_data AS (
    SELECT
        ad.member_id,
        ad.program_id,
        ad.project_id,
        ad.post_id,
        ad.client_id,
        ad.mention_id,
        ad.community_id,
        ad.cost,
        ad.creator_status,
        ad.member_name,
        ad.campaign_name,
        ad.agreement_id,
        ad.date_agreement_agreed_upon,
        ai.id AS agreement_iteration_id,
        ai.price
    FROM agreement_data ad
    LEFT JOIN bi.agreement_iteration ai ON ad.agreement_id = ai.agreement_id
),

-- Step 7: Join with deliverable data to get number of deliverables per agreement_iteration_id
deliverable_data AS (
    SELECT
        ai.agreement_id,
        COUNT(DISTINCT d.id) AS number_of_deliverables
    FROM agreement_iteration_data ai
    LEFT JOIN bi.deliverable d ON ai.agreement_iteration_id = d.agreement_iteration_id
    GROUP BY ai.agreement_id
),

-- Step 8: Combine agreement, agreement_iteration, and deliverable data
combined_agreement_data AS (
    SELECT
        ai.member_id,
        ai.program_id,
        ai.project_id,
        ai.post_id,
        ai.client_id,
        ai.mention_id,
        ai.community_id,
        ai.cost,
        ai.creator_status,
        ai.member_name,
        ai.campaign_name,
        ai.agreement_id,
        ai.date_agreement_agreed_upon,
        ai.price,
        dd.number_of_deliverables,
        CASE
            WHEN dd.number_of_deliverables = 0 THEN ai.price
            ELSE ai.price / dd.number_of_deliverables
        END AS avg_price_per_deliverable,
        CASE
            WHEN dd.number_of_deliverables = 0 THEN ai.price
            ELSE ai.price * dd.number_of_deliverables
        END AS budget_allocated,
        CASE 
            WHEN ai.agreement_id IS NOT NULL THEN
                CASE
                    WHEN dd.number_of_deliverables = 0 THEN ai.price
                    ELSE ai.price * dd.number_of_deliverables
                END
            ELSE 0
        END AS budget_utilized,
        CASE 
            WHEN ai.agreement_id IS NOT NULL THEN 'contract' 
            ELSE 'negotiating' 
        END AS contract_status
    FROM agreement_iteration_data ai
    LEFT JOIN deliverable_data dd ON ai.agreement_id = dd.agreement_id
),

-- Step 9: Join with performance data and social account data
performance_data AS (
    SELECT
        cad.member_id,
        cad.program_id,
        cad.post_id,
        p.link AS post_link,
        p.text AS caption,
        DATE(p.datetime_posted) AS date_posted,
        DATE(p.datetime_posted) AS insight_date,
        p.network, 
        p.post_type,
        SUM(p.impressions) AS impressions,
        SUM(p.likes) AS likes,
        SUM(p.comments) AS comments,
        SUM(p.shares) AS shares,
        SUM(p.favorites) AS favorites,
        SUM(p.plays) AS plays,
        SUM(p.dislikes) AS dislikes,
        p.current_followers AS current_followers,
        SUM(p.closeups) AS closeups,
        SUM(p.estimated_impressions) AS estimated_impressions,
        SUM(p.exits) AS exits,
        p.image AS post_image,
        SUM(p.taps) AS taps,
        SUM(p.unique_impressions) AS unique_impressions,
        SUM(p.views) AS views,
        p.social_account_id AS social_account_id,
        sa.country_code,
        sa.profile_image_url,
        p.datetime_created,
        MIN(DATE(p.datetime_posted)) OVER (PARTITION BY cad.member_id) AS first_activity_date,
        -- Calculate Activation Type
        CASE
            WHEN DATE(p.datetime_posted) = MIN(DATE(p.datetime_posted)) OVER (PARTITION BY cad.member_id) THEN 'New'
            WHEN DATE(p.datetime_posted) > MIN(DATE(p.datetime_posted)) OVER (PARTITION BY cad.member_id) THEN 'Reactivated'
            ELSE NULL
        END AS activation_type,
        -- Calculate Creator Tier
        CASE
            WHEN p.current_followers < 10000 THEN 'Nano'
            WHEN p.current_followers >= 10000 AND p.current_followers < 60000 THEN 'Micro'
            WHEN p.current_followers >= 60000 AND p.current_followers < 200000 THEN 'Mid-tier'
            WHEN p.current_followers >= 200000 THEN 'Macro'
            ELSE 'Not categorized'
        END AS creator_tier,
        -- Calculate Action Date
        COALESCE(p.datetime_posted, MIN(DATE(p.datetime_posted)) OVER (PARTITION BY cad.member_id), p.datetime_created, NOW()) AS action_date,
        cad.member_name,
        cad.creator_status,
        cad.agreement_id,
        cad.date_agreement_agreed_upon,
        cad.price,
        cad.number_of_deliverables,
        cad.avg_price_per_deliverable,
        cad.budget_allocated,
        cad.budget_utilized,
        cad.contract_status,
        cad.campaign_name
    FROM combined_agreement_data cad
    LEFT JOIN bi.post p ON cad.post_id = p.id
    LEFT JOIN bi.social_account sa ON p.social_account_id = sa.id
    GROUP BY 
        cad.member_id,
        cad.program_id,
        cad.post_id,
        p.link,
        p.text,
        p.datetime_posted,
        p.network,
        p.post_type,
        p.current_followers,
        p.image,
        p.social_account_id,
        p.datetime_created,
        cad.member_name,
        cad.creator_status,
        cad.agreement_id,
        cad.date_agreement_agreed_upon,
        cad.price,
        cad.number_of_deliverables,
        cad.avg_price_per_deliverable,
        cad.budget_allocated,
        cad.budget_utilized,
        cad.contract_status,
        cad.campaign_name,
        sa.country_code,
        sa.profile_image_url
)

-- Final Selection
SELECT 
    member_id,
    program_id,
    post_id,
    creator_status,
    social_account_id,
    post_link,
    caption,
    date_posted,
    network,
    post_type,
    impressions,
    likes,
    comments,
    shares,
    favorites,
    plays,
    dislikes,
    current_followers,
    closeups,
    estimated_impressions,
    exits,
    post_image,
    taps,
    unique_impressions,
    views,
    datetime_created,
    first_activity_date,
    activation_type,
    creator_tier,
    action_date,
    member_name,
    number_of_deliverables,
    avg_price_per_deliverable,
    budget_allocated,
    budget_utilized,
    contract_status,
    agreement_id,
    date_agreement_agreed_upon,
    price,
    campaign_name,
    country_code,
    profile_image_url
FROM performance_data;

