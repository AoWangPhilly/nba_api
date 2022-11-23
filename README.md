![Logo](https://www.logodesignlove.com/images/classic/nba-logo.jpg)


# NBA API Project

For this project we want to scrape statistics on all the NBA teams and players from the NBA Advanced Stats webpage


## Table of Contents
1. [Installation](#Installation) 
2. [Getting the NBA Teams](#Getting-the-NBA-Teams) 
3. [Getting the Team Rosters](#Getting-the-Team-Rosters)
4. [Getting All NBA Players](#Getting-All-NBA-Players)
5. [Pre-Processing The Team Roster](#Pre-Processing-The-Team-Roster)
6. [Player Dashboard Statistics](#Player-Dashboard-Statistics)
7. [Player Career Statistics](#Player-Career-Statistics)


## Authors

- [@aowang](https://github.com/AoWangPhilly)
- [@darakasrovi](https://github.com/darakasrovi)

## Acknowledgements

 - [NBA Advanced Stats](https://www.nba.com/stats/help/glossary)
 

## Installation

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
The task for scraping player career statistics takes about 30-45 minutes to execute and is the biggest limitation to our project. Therefore, using Selenium is one of biggest limitations to our project because it requires us to open up the browser to scrape the HTML rather than sending an API request. 

The NBA career stats page does utilize an API, but it requires some header information to make the request. I've experimented with the API (Request URL: https://stats.nba.com/stats/playercareerstats?LeagueID=00&PerMode=Totals&PlayerID=1630178
) and there seems to be a call limit. I tried to gather all the player career stats using the API, but it only allowed me to make 10 requests before buffering endlessly.


## Data Dictionary 
| Field Name   | Data Type | Description                      | Example            |
|--------------|-----------|----------------------------------|--------------------|
| Division     | string    | Name of Division                 | Atlantic           |
| Conference   | string    | Name of Conference               | Western Conference |
| Team         | string    | Name of Team                     | Philadelphia 76ers |
| Team ID      | integer   | ID Associated For Team           | 1610612755         |
| Player ID    | integer   | ID Associated For Player         | 1628379            |
| Player Name  | integer   | Name Associated For Player       | James Harden       |
| Height       | float     | Height Associated For Player     | 203.20	             |
| Weight       | float     | Weight Associated For Player     | 95.25              |
| Birth Date   | datetime  | Birth Date Associated For Player | 1998-03-03         |
| Age          | integer   | Age Associated For Player        | 23                 |
| Exp          | integer   | Years of Experience              | 4                  |
| School       | string    | School the Player Attended       | Duke               |
| PPG          | float     | Points per Game                  | 28.3               |
| RPG          | float     | Rebounds per Game                | 5.6                |
| APG          | float     | Assists per Game                 | 4.3                |
| PIE          | float     | Player Impact Estimate           | 17.4               |
| GP           | integer   | Games Played                     | 45                 |
| MIN          | float     | Minutes Per Game                 | 30.3               |
| FGM          | float     | Field Goals Made per Game        | 11.2               |
| FGA          | float     | Field Goals Attempted per Game   | 21.3               |
| FG%          | float     | Field Goal Percentage per Game   | 45.6%              |
| 3PM          | float     | 3 Pointers Made per Game         | 2.3                |
| 3PA          | float     | 3 Pointers Attempted per Game    | 5.6                |
| 3P%          | float     | 3 Point Percentage per Game      | 35.7%              |
| FTM          | float     | Free Throws Made per Game        | 6.7                |
| FTA          | float     | Free Throws Attempted per Game   | 8.6                |
| FT%          | float     | Free Throw Percentage per Game   | 81.2%              |
| OREB         | float     | Offensive Rebounds per Game      | 4.3                |
| DREB         | float     | Defensive Rebounds per Game      | 5.1                |
| TOV          | float     | Turnovers per Game               | 3.2                |
| STL          | float     | Steals per Game                  | 2.1                |
| BLK          | float     | Blocks per Game                  | 2.2                |
| PF           | float     | Personal Fouls per Game          | 1.9                |
| FP           | float     | Fantasy Points per Game          | 45.3               |
| DD2          | integer   | Double Doubles                   | 4                  |
| TD3          | integer   | Triple Doubles                   | 2                  |
| +/-          | float     | Plus Minus per Game              | 8.4                |
