create table matches(
id int,
city varchar,
date varchar,
player_of_match	varchar,
venue varchar,
neutral_venue int,
team1 varchar,
team2 varchar,
toss_winner varchar,
toss_decision varchar,
winner varchar,
result varchar,
result_margin int,
eliminator varchar,
method varchar,
umpire1	varchar,
umpire2 varchar);


create table deliveries(
	id int,
	inning int,
	over int,
	ball int,
	batsman varchar,
	non_striker varchar,
	bowler varchar,
	batsman_runs int,
	extra_runs int,
	total_runs int,
	is_wicket int,
	dismissal_kind varchar,
	player_dismissed varchar,
	fielder varchar,
	extras_type varchar,
	batting_team varchar,
	bowling_team varchar);
	
copy matches from 'F:\SQL  DATA ANALYTICS\Data_For_Final_ProjectIPLMatches_IPLBall\IPLMatches+IPLBall\IPL_matches.csv'delimiter','csv header;

	
copy deliveries from 'F:\SQL  DATA ANALYTICS\Data_For_Final_ProjectIPLMatches_IPLBall\IPLMatches+IPLBall\IPL_Ball.csv'delimiter','csv header;


select * from matches;
select total_runs from deliveries;
 
select * from matches limit 20;

SELECT * FROM matches WHERE date = '02-05-2013';

select *  from matches where result='runs' and result_margin >100;

select * from matches where result = 'tie' and date ='18-10-2020' order by result  desc;

select count(city) from matches;

CREATE TABLE deliveries_v02 AS
SELECT *, 
       CASE 
           WHEN total_runs >= 4 THEN 'boundary'
           WHEN total_runs = 0 THEN 'dot'
           ELSE 'other'
       END AS ball_result
FROM deliveries;

select count(boundary) from deliveries_v02;
select* from deliveries_v02;

select 
    SUM(CASE WHEN ball_result = 'boundary' THEN 1 ELSE 0 END) AS total_boundaries,
    SUM(CASE WHEN ball_result = 'dot' THEN 1 ELSE 0 END) AS total_dot_balls
FROM deliveries_v02;

SELECT batting_team,
       SUM(CASE WHEN ball_result = 'boundary' THEN 1 ELSE 0 END) AS total_boundaries
FROM deliveries_v02
GROUP BY batting_team
ORDER BY total_boundaries DESC;

SELECT bowling_team,
       SUM(CASE WHEN ball_result = 'dot' THEN 1 ELSE 0 END) AS total_dot_balls
FROM deliveries_v02
GROUP BY bowling_team
ORDER BY total_dot_balls DESC;


SELECT dismissal_kind, COUNT(*) AS total_dismissals
FROM deliveries_v02
WHERE dismissal_kind != 'NA'
GROUP BY dismissal_kind;

select *  from deliveries;

select bowler,extra_runs from deliveries where extra_runs>0 order by extra_runs desc limit 5;



create table deliveries_v03 AS SELECT a.*, b.venue, b.match_date from
deliveries_v02 as a
left join (select max(venue) as venue, max(date) as match_date, id from matches group by
id) as b
on a.id = b.id;

select* from deliveries_v03;

select * from deliveries;

select * from matches;

select venue, sum(total_runs) as total_runs_scored from deliveries_v03 group by venue order by total_runs_scored desc;

SELECT EXTRACT(YEAR FROM CAST(match_date AS DATE)) AS IPL_year,
       SUM(total_runs) AS runs
FROM deliveries_v03
WHERE venue = 'Eden Gardens'
GROUP BY EXTRACT(YEAR FROM CAST(match_date AS DATE))
ORDER BY runs DESC;

-- Step 1: Create the matches_corrected table
CREATE TABLE matches_corrected AS
SELECT *,
       CASE WHEN team1 = 'Rising Pune Supergiants' THEN 'Rising Pune Supergiant' ELSE team1 END AS team1_corr,
       CASE WHEN team2 = 'Rising Pune Supergiants' THEN 'Rising Pune Supergiant' ELSE team2 END AS team2_corr
FROM matches;

select * from matches_corrected;

select * from deliveries_v03;

CREATE TABLE deliveries_v04 AS
SELECT CONCAT(id, '-', inning, '-', over, '-', ball) AS ball_id, *FROM deliveries_v03;

select * from deliveries_v04;

SELECT
    COUNT(*) AS total_rows,
    COUNT(DISTINCT ball_id) AS distinct_ball_ids
FROM deliveries_v04;


CREATE TABLE deliveries_v05 AS
SELECT *,
    ROW_NUMBER() OVER (PARTITION BY ball_id) AS r_num
FROM deliveries_v04;

select * from deliveries_v05;

 select ball_id,r_num from deliveries_v05 WHERE r_num>1;
 
 SELECT *
FROM deliveries_v05
WHERE ball_id IN (
    SELECT ball_id
    FROM deliveries_v05
    WHERE r_num > 1
);
