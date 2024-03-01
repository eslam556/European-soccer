# European soccer

**Author**: Eslam Mohamed Hafez <br>
**Email**: em4132275@gmail.com <br>
**LinkedIn**: https://www.linkedin.com/in/eslam-mohamed-b828521a9

**1.** How many teams in leagues?

````sql
SELECT
    league,
    COUNT(DISTINCT home_team) AS number_of_teams
FROM
    match_view
WHERE
    season = '2015/2016'
GROUP BY
    league,
    season;
````

**Results:**

| league                   | number_of_teams |
|--------------------------|-----------------|
| Belgium Jupiler League   | 16              |
| England Premier League   | 20              |
| France Ligue 1           | 20              |
| Germany 1. Bundesliga    | 18              |
| Italy Serie A            | 20              |
| Netherlands Eredivisie   | 18              |
| Poland Ekstraklasa       | 16              |
| Portugal Liga ZON Sagres | 18              |
| Scotland Premier League  | 12              |
| Spain LIGA BBVA          | 20              |
| Switzerland Super League | 10              |

**2.** How many teams in top five leagues?

````sql
SELECT
    league,
    COUNT(DISTINCT home_team) AS number_of_teams
FROM
    match_view
WHERE
    season = '2015/2016' AND
    league IN ('England Premier League', 'France Ligue 1', 'Germany 1. Bundesliga', 'Italy Serie A', 'Spain LIGA BBVA')
GROUP BY
    league,
    season;
````

**Results:**

| league                 | number_of_teams |
|------------------------|-----------------|
| England Premier League | 20              |
| France Ligue 1         | 20              |
| Germany 1. Bundesliga  | 18              |
| Italy Serie A          | 20              |
| Spain LIGA BBVA        | 20              |


**3.** What is the highest buildup Speed for each team of all time?

````sql
WITH max_speed AS  # Extracting the max speed for each team
(
    SELECT
         team_api_id,
         MAX(buildUpPlaySpeed) AS max_play_speed
    FROM
         team_attributes
    GROUP BY
         team_api_id
    ORDER BY
         team_api_id
),
table_id AS  # Ranking the teams
(
    SELECT
        T.team_long_name,
        TA.date,
        TA.buildUpPlaySpeed AS top_play_speed,
        ROW_NUMBER() OVER(PARTITION BY T.team_long_name ORDER BY TA.date) AS rn
    FROM
        team T
    JOIN
        team_attributes TA
    ON
        T.team_id = TA.team_api_id
    JOIN
        max_speed
    ON
        TA.team_api_id = max_speed.team_api_id
        AND TA.buildUpPlaySpeed = max_speed.max_play_speed
)

SELECT
    team_long_name,
    date,
    top_play_speed
FROM
    table_id
WHERE
    rn = 1  # Avoid redundancy
ORDER BY
    team_long_name,
    date
LIMIT 10;
````

**Results:**

| team_long_name         | date                | top_play_speed |
|------------------------|---------------------|----------------|
| 1. FC Kaiserslautern  | 2014-09-19 00:00:00 | 66             |
| 1. FC Köln             | 2013-09-20 00:00:00 | 60             |
| 1. FC Nürnberg         | 2013-09-20 00:00:00 | 46             |
| 1. FSV Mainz 05        | 2012-02-22 00:00:00 | 68             |
| Aberdeen               | 2010-02-22 00:00:00 | 70             |
| AC Ajaccio             | 2011-02-22 00:00:00 | 65             |
| AC Arles-Avignon       | 2015-09-10 00:00:00 | 56             |
| AC Bellinzona          | 2011-02-22 00:00:00 | 47             |
| Académica de Coimbra  | 2015-09-10 00:00:00 | 53             |
| ADO Den Haag           | 2014-09-19 00:00:00 | 58             |

**4.** Which team has the highest buildup Speed for each league of all time?

````sql
WITH table_id AS  # Ranking the teams
(
SELECT
        T.team_long_name,
        MV.league,
        TA.buildUpPlaySpeed AS top_play_speed,
        ROW_NUMBER() OVER(PARTITION BY MV.league ORDER BY TA.buildUpPlaySpeed DESC) AS rn
    FROM
        team T
    JOIN
        team_attributes TA
    ON
        T.team_id = TA.team_api_id

    JOIN
        match_view MV
    ON
        T.team_long_name = MV.home_team
)
SELECT
    league,
    team_long_name,
    top_play_speed
FROM
    table_id
WHERE
    rn = 1;
````

**Results:**

| league                   | team_long_name           | top_play_speed |
|--------------------------|--------------------------|----------------|
| Belgium Jupiler League   | Standard de Liège        | 75             |
| England Premier League   | West Ham United          | 77             |
| France Ligue 1           | Le Mans FC               | 70             |
| Germany 1. Bundesliga    | Hannover 96              | 78             |
| Italy Serie A            | Carpi                    | 80             |
| Netherlands Eredivisie   | Excelsior                | 73             |
| Poland Ekstraklasa       | Korona Kielce            | 75             |
| Portugal Liga ZON Sagres | FC Porto                 | 70             |
| Scotland Premier League  | Falkirk                  | 70             |
| Spain LIGA BBVA          | CD Numancia              | 71             |
| Switzerland Super League | Grasshopper Club Zürich  | 67             |


**5.** Which team has the highest buildup Speed of all time?

````sql
WITH table_id AS  # Ranking the teams
(
SELECT
        T.team_long_name,
        MV.league,
        TA.buildUpPlaySpeed AS top_play_speed,
        ROW_NUMBER() OVER(PARTITION BY MV.league ORDER BY TA.buildUpPlaySpeed DESC) AS rn
    FROM
        team T
    JOIN
        team_attributes TA
    ON
        T.team_id = TA.team_api_id

    JOIN
        match_view MV
    ON
        T.team_long_name = MV.home_team
)
SELECT
    team_long_name
FROM
    table_id
WHERE
    rn = 1
ORDER BY
    top_play_speed DESC
LIMIT 1;
````

**Results:**

| team_long_name | 
|----------------|
| Carpi          |


**6.** How many goals scored by each team for each season?

````sql
WITH goals_per_club AS
(
SELECT  # Displaying teams and their goals as home_team
    home_team,
    season,
    SUM(home_team_goal) AS total_home_goals
FROM
    match_view
GROUP BY
    home_team,
    season

UNION ALL

SELECT  # Displaying teams and their goals as away_team
    away_team,
    season,
    SUM(away_team_goal) AS total_away_goals
FROM
    match_view
GROUP BY
    away_team,
    season
)

SELECT
    home_team AS team,
    season,
    SUM(total_home_goals) AS total_goals
FROM
    goals_per_club
GROUP BY
    home_team,
    season
ORDER BY
    team,
    season
LIMIT 10;
````

**Results:**

| team               | season    | total_goals |
|--------------------|-----------|-------------|
| 1. FC Kaiserslautern | 2010/2011 | 48          |
| 1. FC Kaiserslautern | 2011/2012 | 24          |
| 1. FC Köln         | 2008/2009 | 35          |
| 1. FC Köln         | 2009/2010 | 33          |
| 1. FC Köln         | 2010/2011 | 47          |
| 1. FC Köln         | 2011/2012 | 39          |
| 1. FC Köln         | 2014/2015 | 34          |
| 1. FC Köln         | 2015/2016 | 38          |
| 1. FC Nürnberg     | 2009/2010 | 32          |
| 1. FC Nürnberg     | 2010/2011 | 47          |


**7.** What is the maximum number of goals scored for each team?

````sql
WITH goals_per_club AS
(
SELECT  # Displaying teams and their goals as home_team
    home_team,
    season,
    SUM(home_team_goal) AS total_home_goals
FROM
    match_view
GROUP BY
    home_team,
    season

UNION ALL

SELECT  # Displaying teams and their goals as away_team
    away_team,
    season,
    SUM(away_team_goal) AS total_away_goals
FROM
    match_view
GROUP BY
    away_team,
    season
),
total_goals AS
(
SELECT  # DISPLAYING teams and their total goals each season
    home_team AS team,
    season,
    SUM(total_home_goals) AS total_goals,
    ROW_NUMBER() OVER(PARTITION BY home_team ORDER BY SUM(total_home_goals) DESC) AS rn
FROM
    goals_per_club
GROUP BY
    home_team,
    season
ORDER BY
    team,
    season
)

SELECT
    team,
    season,
    total_goals AS max_goals
FROM
    total_goals
WHERE
    rn = 1  # Getting the max number of goals for each team
GROUP BY
    team,
    season,
    total_goals
ORDER BY
    team,
    season
LIMIT 10;
````

**Results:**

| team                   | season    | max_goals |
|------------------------|-----------|-----------|
| 1. FC Kaiserslautern  | 2010/2011 | 48        |
| 1. FC Köln            | 2010/2011 | 47        |
| 1. FC Nürnberg        | 2010/2011 | 47        |
| 1. FSV Mainz 05       | 2013/2014 | 52        |
| Aberdeen              | 2015/2016 | 62        |
| AC Ajaccio            | 2011/2012 | 40        |
| AC Arles-Avignon      | 2010/2011 | 21        |
| AC Bellinzona         | 2008/2009 | 44        |
| Académica de Coimbra | 2009/2010 | 37        |
| ADO Den Haag          | 2010/2011 | 63        |


**8.** Who is the team with most match wins for each league?

````sql
WITH match_score AS
(
SELECT  # Displaying teams and their goals as home_team
    id,
    home_team AS team,
    CASE
        WHEN home_team_goal > away_team_goal THEN 1 ELSE 0 END AS Winning_match
FROM
    match_view

UNION ALL

SELECT  # Displaying teams and their goals as away_team
    id,
    away_team AS team,
    CASE
        WHEN away_team_goal > home_team_goal THEN 1 ELSE 0 END AS Winning_match
FROM
    match_view
),
winning_matches AS
(
SELECT  # Displaying total match wins for each team
    MV.league,
    M.team,
    COUNT(CASE WHEN M.Winning_match = 1 THEN 1 END) AS wins,
    ROW_NUMBER() OVER(PARTITION BY MV.league ORDER BY COUNT(CASE WHEN M.Winning_match = 1 THEN 1 END) DESC) AS rn
FROM
    match_score M
JOIN
    match_view MV
ON
    M.id = MV.id
GROUP BY
    MV.league,
    team
ORDER BY
    league,
    wins DESC
)

SELECT
    league,
    team,
    MAX(wins) AS match_wins
FROM
    winning_matches
WHERE
    rn = 1  # Getting the max number of winning matchs for each team
GROUP BY
    league,
    team
ORDER BY
    league;
````

**Results:**

| league                   | team               | match_wins |
|--------------------------|--------------------|------------|
| Belgium Jupiler League   | RSC Anderlecht     | 136        |
| England Premier League   | Manchester United  | 192        |
| France Ligue 1           | Paris Saint-Germain| 175        |
| Germany 1. Bundesliga    | FC Bayern Munich   | 193        |
| Italy Serie A            | Juventus           | 189        |
| Netherlands Eredivisie   | Ajax               | 181        |
| Poland Ekstraklasa       | Legia Warszawa     | 137        |
| Portugal Liga ZON Sagres | SL Benfica         | 185        |
| Scotland Premier League  | Celtic             | 218        |
| Spain LIGA BBVA          | FC Barcelona       | 234        |
| Switzerland Super League | FC Basel           | 180        |


**9.** Who is the best stricker of the season for each team?

````sql
WITH best_players AS
(
SELECT  # Displaying the players with their rating
    M.league,
    M.season,
    M.home_team AS team,
    M.home_second_forward AS player,
    PA.overall_rating,
    ROW_NUMBER() OVER(PARTITION BY M.home_team, M.season ORDER BY PA.overall_rating DESC) AS rn
FROM
    match_view M
JOIN
    player P
ON
    P.player_name = M.home_second_forward
JOIN
    player_attributes PA
ON
    P.player_id = PA.player_api_id
)

SELECT
    league,
    season,
    team,
    player,
    overall_rating
FROM
    best_players
WHERE
    rn = 1  # Getting the best stricker of the season for each team
LIMIT 10;
````

**Results:**

| league                 | season    | team                | player            | overall_rating |
|------------------------|-----------|---------------------|-------------------|----------------|
| Germany 1. Bundesliga | 2010/2011 | 1. FC Kaiserslautern | Ivo Ilicevic      | 77             |
| Germany 1. Bundesliga | 2011/2012 | 1. FC Kaiserslautern | Olcay Sahan       | 79             |
| Germany 1. Bundesliga | 2008/2009 | 1. FC Köln           | Sergiu Marian Radu| 79             |
| Germany 1. Bundesliga | 2009/2010 | 1. FC Köln           | Lukas Podolski    | 87             |
| Germany 1. Bundesliga | 2010/2011 | 1. FC Köln           | Lukas Podolski    | 87             |
| Germany 1. Bundesliga | 2011/2012 | 1. FC Köln           | Lukas Podolski    | 87             |
| Germany 1. Bundesliga | 2014/2015 | 1. FC Köln           | Anthony Ujah       | 76             |
| Germany 1. Bundesliga | 2015/2016 | 1. FC Köln           | Anthony Modeste    | 78             |
| Germany 1. Bundesliga | 2009/2010 | 1. FC Nürnberg       | Marek Mintal      | 83             |
| Germany 1. Bundesliga | 2010/2011 | 1. FC Nürnberg       | Mehmet Ekici      | 78             |


**10.** Who is the player with most wins?

````sql
CREATE VIEW winning_players AS  # Displaying all players with winning matches
(
WITH players AS
(
    SELECT home_gk AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_center_back_1 AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_center_back_2 AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_right_back AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_left_back AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_1 AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_2 AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_3 AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_4 AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_second_forward AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT home_center_forward AS player 
    FROM match_view 
    WHERE home_team_goal > away_team_goal
    
    UNION ALL
    
    SELECT away_gk AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_center_back_1 AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_center_back_2 AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_right_back AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_left_back AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_1 AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_2 AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_3 AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_4 AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_second_forward AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
    
    UNION ALL
    
    SELECT away_center_forward AS player 
    FROM match_view 
    WHERE away_team_goal > home_team_goal
)


SELECT
    player,
    COUNT(*) AS total_wins
FROM
    players
GROUP BY
    player
);

SELECT
    *
FROM
    winning_players
WHERE
    player IS NOT NULL
ORDER BY
    total_wins DESC
LIMIT 1;
````

**Results:**

| player  | total_wins |
|---------|------------|
| Marcelo | 301        |


**11.** Who is the player with most losses?

````sql
CREATE VIEW lost_players AS  # Displaying all players with lost matches
(
WITH players AS
(
    SELECT home_gk AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_center_back_1 AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_center_back_2 AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_right_back AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_left_back AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_1 AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_2 AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_3 AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_4 AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_second_forward AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT home_center_forward AS player 
    FROM match_view 
    WHERE home_team_goal < away_team_goal
    
    UNION ALL
    
    SELECT away_gk AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_center_back_1 AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_center_back_2 AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_right_back AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_left_back AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_1 AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_2 AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_3 AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_4 AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_second_forward AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
    
    UNION ALL
    
    SELECT away_center_forward AS player 
    FROM match_view 
    WHERE away_team_goal < home_team_goal
)

SELECT
    player,
    COUNT(*) AS total_losses
FROM
    players
GROUP BY
    player
);

SELECT
    *
FROM
    lost_players
WHERE
    player IS NOT NULL
ORDER BY
    total_losses DESC
LIMIT 1;
````

**Results:**

| player  | total_losses |
|---------|--------------|
| Ricardo | 163          |


**12.** Who is the player with most draws?

````sql
CREATE VIEW draw_players AS  # Displaying all players with draw matches
(
WITH players AS
(
    SELECT home_gk AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_center_back_1 AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_center_back_2 AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_right_back AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_left_back AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_1 AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_2 AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_3 AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_midfield_4 AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_second_forward AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT home_center_forward AS player 
    FROM match_view 
    WHERE home_team_goal = away_team_goal
    
    UNION ALL
    
    SELECT away_gk AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_center_back_1 AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_center_back_2 AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_right_back AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_left_back AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_1 AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_2 AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_3 AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_midfield_4 AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_second_forward AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
    
    UNION ALL
    
    SELECT away_center_forward AS player 
    FROM match_view 
    WHERE away_team_goal = home_team_goal
)

SELECT
    player,
    COUNT(*) AS total_draws
FROM
    players
GROUP BY
    player
);

SELECT
    *
FROM
    draw_players
WHERE
    player IS NOT NULL
ORDER BY
    total_draws DESC
LIMIT 1;
````

**Results:**

| player  | total_draws |
|---------|-------------|
| Ricardo | 129         |


**13.** Which match has the highest score in each season?

````sql
SELECT
    match_api_id,
    season,
    league,
    home_team,
    away_team,
    MAX(home_team_goal) AS home_team_goal,
    MIN(away_team_goal) AS away_team_goal
FROM
    match_view
GROUP BY
    match_api_id,
    season,
    league,
    home_team,
    away_team
ORDER BY
    home_team_goal DESC,
    away_team_goal
LIMIT 2;
````

**Results:**

| match_api_id | season    | league              | home_team       | away_team       | home_team_goal | away_team_goal |
|--------------|-----------|---------------------|-----------------|-----------------|----------------|----------------|
| 836306       | 2010/2011 | Netherlands Eredivisie | PSV           | Feyenoord       | 10             | 0              |
| 2030233      | 2015/2016 | Spain LIGA BBVA        | Real Madrid CF | Rayo Vallecano | 10             | 2              |


**Player stats**

````sql
SELECT
    P.player_id,
    P.player_name AS name,
    TIMESTAMPDIFF(YEAR, P.birthday, NOW()) AS age,
    SUM(W.total_wins + D.total_draws + L.total_losses) AS total_matches,
    G.number_of_goals,
    W.total_wins AS win,
    D.total_draws AS draw,
    L.total_losses AS lose
FROM
    player P
JOIN
    player_goals G
ON
    P.player_id = G.id
JOIN
    winning_players W
ON
    W.player = P.player_name
JOIN
    draw_players D
ON
    D.player = P.player_name
JOIN
    lost_players L
ON
    L.player = P.player_name
GROUP BY
    P.player_id,
    P.player_name,
    TIMESTAMPDIFF(YEAR, P.birthday, NOW()),
    G.number_of_goals,
    W.total_wins,
    D.total_draws,
    L.total_losses
ORDER BY
    number_of_goals DESC,
    total_matches
LIMIT 10;
````

**Results:**

| player_id | name               | age | total_matches | number_of_goals | win  | draw | lose |
|-----------|--------------------|-----|---------------|-----------------|------|------|------|
| 30981     | Lionel Messi       | 36  | 249           | 295             | 194  | 33   | 22   |
| 30893     | Cristiano Ronaldo  | 39  | 259           | 290             | 199  | 33   | 27   |
| 35724     | Zlatan Ibrahimovic | 42  | 235           | 207             | 166  | 46   | 23   |
| 25759     | Gonzalo Higuain    | 36  | 205           | 174             | 141  | 32   | 32   |
| 49677     | Edinson Cavani     | 37  | 239           | 168             | 133  | 56   | 50   |
| 27734     | Antonio Di Natale  | 46  | 215           | 156             | 94   | 51   | 70   |
| 37412     | Sergio Aguero      | 35  | 221           | 156             | 128  | 38   | 55   |
| 40636     | Luis Suarez        | 37  | 224           | 136             | 137  | 41   | 46   |
| 30829     | Wayne Rooney       | 38  | 223           | 135             | 142  | 40   | 41   |
| 30843     | Robin van Persie   | 40  | 172           | 124             | 95   | 44   | 33   |