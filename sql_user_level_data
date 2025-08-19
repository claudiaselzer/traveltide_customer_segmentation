/*
  Query: user_level_data 
  Description: Builds user-level profiles combining demographics, 
               session behavior, booking activity, and loyalty metrics.
  Usage: Used to prepare features for clustering and customer segmentation.
  Author: Claudia
  Date: 2025-08-19
*/




-- Filter sessions within the past 12 months
WITH filtered_sessions AS (
    SELECT *
    FROM sessions
    WHERE session_start BETWEEN '2023-07-29' AND '2023-07-29'
),

-- Only include users who made at least one booking
booked_users AS (
    SELECT DISTINCT user_id
    FROM filtered_sessions
    WHERE flight_booked = TRUE OR hotel_booked = TRUE
),

-- Create user-level profile with demographic and behavioral metrics
user_profile AS (
    SELECT
        u.user_id,
        u.gender,
        u.birthdate,
        DATE_PART('year', AGE(u.birthdate)) AS age,

        -- Age group segmentation
        CASE
            WHEN DATE_PART('year', AGE(u.birthdate)) < 18 THEN 'Teenager (<18)'
            WHEN DATE_PART('year', AGE(u.birthdate)) BETWEEN 18 AND 24 THEN 'Young Adults (18-24)'
            WHEN DATE_PART('year', AGE(u.birthdate)) BETWEEN 25 AND 34 THEN 'Early Career (25-34)'
            WHEN DATE_PART('year', AGE(u.birthdate)) BETWEEN 35 AND 49 THEN 'Established (35-49)'
            WHEN DATE_PART('year', AGE(u.birthdate)) BETWEEN 50 AND 64 THEN 'Prime Leisure (50-64)'
            ELSE 'Senior Explorer (65+)'
        END AS age_group,

        -- Customer lifetime (in months) until July 2023
        DATE_PART('year', AGE(DATE '2023-07-29', u.sign_up_date)) * 12 +
        DATE_PART('month', AGE(DATE '2023-07-29', u.sign_up_date)) AS customer_since_months,

        -- Customer loyalty category
        CASE 
            WHEN (DATE_PART('year', AGE(DATE '2023-07-29', u.sign_up_date)) * 12 +
                  DATE_PART('month', AGE(DATE '2023-07-29', u.sign_up_date))) < 6 THEN 'New (<6 months)'
            WHEN (DATE_PART('year', AGE(DATE '2023-07-29', u.sign_up_date)) * 12 +
                  DATE_PART('month', AGE(DATE '2023-07-29', u.sign_up_date))) < 24 THEN 'Established (6–24 months)'
            ELSE 'Loyal (>2 years)'
        END AS customer_lifetime_group,

        -- Session behavior
        COUNT(s.session_id) AS total_sessions,
        ROUND(AVG(EXTRACT(EPOCH FROM (s.session_end - s.session_start)) / 60), 2) AS avg_session_minutes,
        ROUND(SUM(s.page_clicks)::decimal / COUNT(s.session_id), 2) AS avg_clicks_per_session,

        -- Booking behavior
        SUM(CASE WHEN s.flight_booked THEN 1 ELSE 0 END) AS total_flight_bookings,
        SUM(CASE WHEN s.hotel_booked THEN 1 ELSE 0 END) AS total_hotel_bookings,
        SUM(CASE WHEN s.flight_booked AND s.hotel_booked THEN 1 ELSE 0 END) AS total_flight_hotel_bookings,

        -- Conversion
        SUM(CASE WHEN s.flight_booked OR s.hotel_booked THEN 1 ELSE 0 END) AS converted_sessions,
        ROUND(100.0 * SUM(CASE WHEN s.flight_booked OR s.hotel_booked THEN 1 ELSE 0 END) / COUNT(s.session_id), 2) AS conversion_rate_percent,

        -- Cancellation behavior
        SUM(CASE WHEN s.cancellation THEN 1 ELSE 0 END) AS total_cancellations,
        ROUND(100.0 * SUM(CASE WHEN s.cancellation THEN 1 ELSE 0 END) / COUNT(s.session_id), 2) AS cancellation_rate_percent,

        -- Discount usage
        SUM(COALESCE(s.flight_discount_amount, 0) + COALESCE(s.hotel_discount_amount, 0)) AS total_discount_used,
        CASE 
            WHEN SUM(COALESCE(s.flight_discount_amount, 0) + COALESCE(s.hotel_discount_amount, 0)) > 0 THEN TRUE
            ELSE FALSE
        END AS used_discount,

        -- Demographic info
        u.married,
        u.has_children,
        u.home_country,
        u.home_city,
        u.sign_up_date

    FROM users u
    JOIN filtered_sessions s ON u.user_id = s.user_id
    JOIN booked_users b ON u.user_id = b.user_id
    GROUP BY u.user_id, u.gender, u.birthdate, u.married, u.has_children, u.home_country, u.home_city, u.sign_up_date
)

-- Step 4: Final result
SELECT *
FROM user_profile;
