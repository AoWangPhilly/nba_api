![Logo](https://www.logodesignlove.com/images/classic/nba-logo.jpg)

# NBA API Project

For this project we want to scrape data on all the NBA teams and players from the NBA Advanced Stats webpage. The scripts will scrape all the NBA teams and team rosters for the 2022-23 season. Additionally, the script will gather the career regular season stats for each player.

After the data acquisition step, we have many ways for data distribution. The script automatically saves each piece of data into CSV files, so we could upload them to Kaggle. Additionally, we also developed and [hosted](https://dsci511-nba-api.herokuapp.com/docs) an API, so it'll be easier for programmers to get the data.

## Table of Contents
1. [Installation](#Installation) 
2. [Getting the NBA Teams](#Getting-the-NBA-Teams) 
3. [Getting the Team Rosters](#Getting-the-Team-Rosters)
4. [Getting All NBA Players](#Getting-All-NBA-Players)
5. [Pre-Processing The Team Roster](#Pre-Processing-The-Team-Roster)
6. [Player Dashboard Statistics](#Player-Dashboard-Statistics)
7. [Player Career Statistics](#Player-Career-Statistics)
8. [Limitations](#limitations)
9. [Data Dictionary](#data-dictionary)
10. [API Creation](#api-creation)
11. [API Endpoints](#api-endpoints)


## Authors

- [@aowang](https://github.com/AoWangPhilly)
- [@darakasrovi](https://github.com/darakasrovi)

## Acknowledgements

 - [NBA Advanced Stats](https://www.nba.com/stats/help/glossary)
 

## Installation
The project primarily needs the `requests`, `BeautifulSoup`, and `Selenium` packages to webscraping and `pandas` for data cleaning. We also used `FastAPI` for developing the API and `SQLAlchemy` to write queries for Postgres. Below is the bash command to install all necessary packages to run the Jupyter Notebook and API.

```bash
pip install -r requirements.txt
```

## Getting The NBA Teams
The first part of our notebook is getting the data of all the NBA Teams. The function goes to the NBA team stats page and scrapes all the team data. This includes the division, team, and team ID. We used BeautifulSoup and regex to get the table of teams. 

```python
soup = BeautifulSoup(response.content, "html.parser")
regex = re.compile("^StatsTeamsList_divContent")
table = soup.find("div", {"class": regex})
```
And we were able to get the division, team, and team ID in the HTML from the website.
## Getting The Team Rosters
The second part of our notebook is getting the team rosters. We wanted to get all the teams rosters for this current NBA season. We created a function that helps you create the URL given the team ID. We ran a while loop that kept on calling until we got our desired data. 
```python
 i = 0
 while not response.ok:
    print(f"There was an issue getting team id={team_id}!!")
    print(f"Reattempting! Iteration {i + 1}")
    i += 1
    response = requests.get(url)
```

Sometimes when we called the URL it would freeze up on us or would refuse to give us data so that is why we used a while loop. After doing this we were able to get the correct roster data.
## Getting All NBA Players
The third part of our notebook is getting all the players in the NBA. In order to do this we needed to create a list of team ID's. We entered it in the function in order to get the team rosters from all the NBA teams. Then we created a thread in order to complete the various tasks more efficiently and to increase the speed of our program. Instead of waiting to get one team roster at a time and for each task to finish, we can use a thread to get all the team rosters at the same time.
```python
with futures.ThreadPoolExecutor() as executor:
    player_list = list(executor.map(get_team_roster, team_ids))
return pd.concat(player_list).reset_index(drop=True)
```

## Pre-Processing The Team Roster
The fourth part of our notebook is pre-processing the team roster data. We converted feet/inches to meter/centimeter. We also converted pounds to kilograms. Then we dropped the following columns: LeagueID, NICKNAME, PLAYER_SLUG, HOW_ACQUIRED. Additionally, we converted the Birth Date to a DateTime object. We did this because the way it was before would be a string. If we wanted to find players who were born a certain year, month, or day, it would be irritating dealing with strings. By converting to datetime, it allows us to do more filtering.  
## Player Dashboard Statistics
The fifth part of our notebook is getting player dashboard statistics. These stats consist of Points Per Game (PPG) Rebounds Per Game (RPG) Assists Per Game (APG) and the Player Impact Estimate (PIE). This function is very similar to the team roster however the one thing changes is the player_id instead of team_id. We then used that URL to get the "Quick Stats" which are PPG, RPG, APG, and PIE.
## Player Career Statistics
The final part of our project was getting the player career statistics. This was where we needed to use selenium as the data is dynamic and constantly changing. The first function checks when the loading screen stops and the data is ready. The second function gets the actual stats. We were looking at career regular season stats for each player. Then we had to collect and format the data and put it inside a dataframe and return it. In some cases, players did not have career stats so we decided to mention that there is no data available for such cases.
```python
if soup.find("div", string="No data available"):
    print(f"No data available for player: {player_id}")
    return pd.DataFrame()
print("There seems to be another issue!!")
 ```        
These players are usually rookies or reserve players that do not get play time. We then had to run a for loop to get all the player's career stats.
```python
output = []
for idx, player_id in enumerate(player_ids):
    print(f"#{idx}", end=" ")
    output.append(get_player_info(player_id))
return pd.concat(output).reset_index(drop=True)
 ```

## Limitations
The task for scraping player career statistics takes about 30-45 minutes to execute and is the biggest limitation to our project. Because we use Selenium, it requires us to open up the browser to scrape the HTML rather than sending an API request, which would generally be much quicker.

The NBA career stats page does utilize an API, but it requires some header information to make the request. I've experimented with the API (Request URL: https://stats.nba.com/stats/playercareerstats?LeagueID=00&PerMode=Totals&PlayerID=1630178
) and there seems to be a call limit. I attempted to gather all the player career stats using the API, but it only allowed me to make 10 requests before buffering endlessly.


## Data Dictionary 
| Field Name   | Data Type | Description                      | Example            |
|--------------|-----------|----------------------------------|--------------------|
| Division     | string    | Name of Division                 | Atlantic           |
| Team         | string    | Name of Team                     | Philadelphia 76ers |
| Team ID      | integer   | ID Associated For Team           | 1610612755         |
| Team Abbreviation| string| Abbreviation of Team Name        | PHL
| Season       | integer   | One year in which regulated games of the sport are in session           | 2022
| Player ID    | integer   | ID Associated For Player         | 1628379            |
| Player Name  | string   | Name Associated For Player       | James Harden       |
| Player Number| integer   | The number worn on a player's uniform | #1
| Position     | string    | The role the player plays in the game | G
| Height       | float     | Height Associated For Player     | 196.0	           |
| Weight       | float     | Weight Associated For Player     | 100                |
| Birth Date   | datetime  | Birth Date Associated For Player | 1989-08-26         |
| Age          | integer   | Age Associated For Player        | 33                 |
| Exp          | string    | Years of Experience              | 13 or R            |
| School       | string    | School the Player Attended       | Arizona State      |
| PPG          | float     | Points per Game                  | 22.0               |
| RPG          | float     | Rebounds per Game                | 7.0                |
| APG          | float     | Assists per Game                 | 10.0               |
| PIE          | float     | Player Impact Estimate           | 17.4               |
| GP/GS        | integer   | Games Played                     | 9                  |
| MIN          | integer   | Minutes Per Game                 | 331	               |
| PTS          | integer   | The number of points scored      | 198	               |
| FGM          | integer   | Field Goals Made                 | 63                 |
| FGA          | integer   | Field Goals Attempted            | 143                |
| FG%          | float     | Field Goal Percentage            | 44.1               |
| 3PM          | integer   | 3 Point Field Goals Made         | 20                 |
| 3PA          | integer   | 3 Pointers Attempted             | 60                 |
| 3P%          | float     | 3 Point Percentage               | 33.3               |
| FTM          | integer   | Free Throws Made                 | 52                 |
| FTA          | integer   | Free Throws Attempted            | 5 6                |
| FT%          | float     | Free Throw Percentage            | 92.9               |
| OREB         | integer   | Offensive Rebounds               | 5                  |
| DREB         | integer   | Defensive Rebounds               | 58                 |
| REB          | integer   | Rebounds                         | 63                 |
| AST          | integer   | The number of assists -- passes that lead directly to a made basket -- by a player               | 90 |
| STL          | integer   | Number of times a defensive player or team takes the ball from a player on offense, causing a turnover | 10 |
| BLK          | integer   | The number of shots attempted by a player or team that are blocked by a defender  | 6 |
| TOV          | integer   | A turnover occurs when the player or team on offense loses the ball to the defense  | 26 
| PF           | integer   | The number of personal fouls a player or team committed | 18

## API Creation
We decided to develop an API because it's one of the goals we set out for the project scope. All code for the API is in the `api` directory and the `main.py` file in the base directory. We also decided to develop it with FastAPI, since it provides a nice documentation page showcasing each endpoint.

The first step for creating the API is creating the database. In `main.py`, we create the NBA database, then we read in the CSV files saved from the previous execution as data frames and save them as database tables. 

Next, we need some way to make queries to the database tables. In `api/database.py`, we created a method, `get_db`, which creates a local session using SQLAlchemy. From there, we also need a way to model the tables to make querying simpler. In `api/models.py`, we created a model for each table.

Finally, the endpoints are all created in `main.py`.

## API Endpoints
* Get all NBA Teams
    - https://dsci511-nba-api.herokuapp.com/teams
* Get NBA Team Roster
    - https://dsci511-nba-api.herokuapp.com/roster/TEAM_ID
* Get Player General Information
    - https://dsci511-nba-api.herokuapp.com/player/PLAYER_ID
* Get Player Quick Stats
    - https://dsci511-nba-api.herokuapp.com/player/quickstats/PLAYER_ID
* Get Player Career Stats
    - https://dsci511-nba-api.herokuapp.com/player/careerstats/PLAYER_ID
