# Naukri-Resume-Sql-Case-study

-- count the no of users per month in the year 2024?


select count(user_id) as no_of_users , date_format(registration_datetime, "%Y-%m") as month 
from user_registration
where year(registration_datetime) = 2024
group by date_format(registration_datetime, "%Y-%m");

-- what is the count of registration every month on the "Resume Now" portal for 2024 ?


SELECT 
    DATE_FORMAT(u.registration_datetime, "%Y-%m") AS month,
    COUNT(u.user_id) AS no_of_users
FROM portal p
inner JOIN  user_registration u ON u.portal_id = p.portal_id
WHERE p.portal_name = "Resume Now"
  AND YEAR(registration_datetime) = 2024
GROUP BY DATE_FORMAT(u.registration_datetime, "%Y-%m")
ORDER BY month;

-- ques 2 : which portal has highest subscription rate for users registered in the last 30 days ?
-- subscription rate = total subscription / total registration. 

select portal_id, count(*) as total_registration, count(subscription_datetime) as total_subscription,
round(((count(subscription_datetime)*100.0)/count(*)),2) as subscription_rate
from user_registration
group by portal_id;

select p.portal_name, -- count(*) as total_registration, count(u.subscription_datetime) as total_subscription,
round(((count(u.subscription_datetime)*100.0)/count(*)),2) as subscription_rate
from user_registration u inner join portal p 
using(portal_id)
-- where datediff(curdate(), u.registration_datetime) <= 30
group by p.portal_name
order by subscription_rate desc 
limit 1;

-- how many registered users create less than 3 resumes ?

with cte as (
select count(r.resume_id) as no_of_resumes, u.user_id 
from user_registration u left join resume_doc r 
using(user_id)
group by u.user_id )
select * from cte
where no_of_resumes <3;


-- or -- 

select count(r.resume_id) as no_of_resumes, u.user_id 
from user_registration u left join resume_doc r 
using(user_id)
group by u.user_id
having count(r.resume_id) <3;

-- create a list of users who subscribed in 2024 on the 'Zety' portal and get the experience_years on the first resume?


select u.*, p.portal_code
 from user_registration u join portal p 
 using(portal_id)
where year(u.subscription_datetime)=2024 and p.portal_code = 'ZETY';

with cte as (
select u.user_id, r.experience_years, u.subscription_datetime, p.portal_code, row_number() over (partition by user_id order by user_id) as rn
from resume_doc r  join  user_registration u 
using(user_id) join portal p using(portal_id))
select user_id,experience_years from cte
where portal_code = 'ZETY' and rn =1 and experience_years > 0;
