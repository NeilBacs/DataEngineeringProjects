# Data Extraction in Python


#### Import the necessary libraries , made for easier way to get to the endpoints and the function names are very descriptive
```python
from chessdotcom import Client, get_player_profile,get_country_players, get_club_members, get_club_details, get_country_clubs, get_player_stats
import pandas as pd
import numpy as np
```

#### Need this to call API based on documentation

```python
Client.request_config["headers"]["User-Agent"] = (
   "For Portfolio Project. contact: leoneilbacon@gmail.com" 
)
```


#### Initially, I want to get all player data on all accounts. However the only function we have is get_country_players. I also tried it first but it actually truncates its result and only returns 10000 players and It have not reach even reach my account
```python
print(get_country_players("PH"))
```


#### Now I decided to get the data of members  of one club using get_club_members, and the club I chose is where I am a member and also the largest group in my country
```python
club = get_club_members("philippines-finest-chess-club").json
```

#### Examine the contents of the json file
```python
print(club)
print(club.keys())
print(club["members"].keys())
```

#### Under member key, there are 3 keys corresponds to weekly, monthly, and all time based on member's activity and it contains the username and joined data which is integer, this may be corresponds to date and time but in integer format.
#### My concern is the username so I iterate each contents using for loop

```python
print("Weekly")
print("_________________________________________________________")
for i in club["members"]["weekly"]:
   print (i["username"])

print("Monthly")
print("_________________________________________________________")
for i in club["members"]["monthly"]:
  print (i["username"])

print("All_time")
print("_________________________________________________________")
for i in club["members"]["all_time"]:
   print (i["username"])
```

#### I combine the members in weeekly, monthly, and all_time and  compare its lenght to the displayed number of members on the club using get_club_details function
```python
club_members = club["members"]["all_time"] + club["members"]["monthly"] + club["members"]["weekly"]
club_details = get_club_details("philippines-finest-chess-club").json
print(f'Club members sum : {len(club_members)} | Club details : {club_details["club"]["members_count"]}')
```

#### The result matches, if not, the reason might be the delayed update on club details

#### Now let me first store this club_members in a data frame to have it sturctured in rows and column format. Can easily done by pandas library

```python
df_club_members = pd.DataFrame(club_members)
df_club_members
```

#### Just to confirm that the username column has no duplicates
```python
print(f"Number of rows: {df_club_members.shape[0]}| Number of  unique username: {df_club_members['username'].nunique()}")
```

#### Convert joined date from unix timestamp to actualreadable date
```python
df_club_members['joined'] = pd.to_datetime(df_club_members['joined'], unit='s') 
df_club_members
```

#### Now let's get more data about the members using different functions. Particularly their stats. So we will use get_player_stats function
```python
#let me test using my username first
my_stat = get_player_stats("yschiewnthia").json
my_stat
```



#### My interest is the ches rapid last rating and tactics highest rating
```python
my_stat['stats']['chess_rapid']['last']['rating']
my_stat['stats']['tactics']['highest']['rating']
```
#### Since get_player_stats only accepts one username, we need to pass each username of the members one by one using for loop
#### However , I experienced limit error to the for loop below so partioning them to 11 and just make the for loop each of the parts

```python
batch1= club_members[:2000]
batch2= club_members[2000:4000]
batch3= club_members[4000:6000]
batch4= club_members[6000:8000]
batch5= club_members[8000:10000]
batch6= club_members[10000:12000]
batch7= club_members[12000:14000]
batch8= club_members[14000:16000]
batch9= club_members[16000:18000]
batch10= club_members[18000:20000]
batch11 = club_members[20000:]



#initializing the dictionaries that will hold the return of get_player_stats on each username in respective batches
batch1_player_stats_dicts=[] 
batch2_player_stats_dicts=[] 
batch3_player_stats_dicts=[] 
batch4_player_stats_dicts=[] 
batch5_player_stats_dicts=[] 
batch6_player_stats_dicts=[] 
batch7_player_stats_dicts=[] 
batch8_player_stats_dicts=[] 
batch9_player_stats_dicts=[] 
batch10_player_stats_dicts=[] 
batch11_player_stats_dicts=[] 
```
#### This is the for loop

```python
for member in batch11: # change here
    
    # if username got deleted or renamed and can't be found now. It will be skipped
    try: 
        member_stat = get_player_stats(member['username']).json
    except:
        continue

    player_stats_dict={}
    

    player_stats_dict["username"] = member['username']
    
    try:
        player_stats_dict["rapid_rating"] = member_stat['stats']['chess_rapid']['last']['rating']
    except:
        player_stats_dict["rapid_rating"] =  None 
    try:
        player_stats_dict["tactics_rating"] = member_stat['stats']['tactics']['highest']['rating']
    except:
        player_stats_dict["tactics_rating"] = None
    
    batch11_player_stats_dicts.append(player_stats_dict) # change here
```

#### Combine batches
 ```python
batch1_to_11 = batch1_player_stats_dicts +  batch2_player_stats_dicts +  batch3_player_stats_dicts +  batch4_player_stats_dicts +  batch5_player_stats_dicts+ batch6_player_stats_dicts +  batch7_player_stats_dicts +  batch8_player_stats_dicts +  batch9_player_stats_dicts +  batch10_player_stats_dicts + batch11_player_stats_dicts
```

#### Transform lists of dictionaries to df
 ```python
df_stats_members = pd.DataFrame(batch1_to_11)
```
    
#### Validate if equal length
 ```python
print(len(df_club_members) == len(df_stats_members))

# If It is false , possible reason is there might be records of users in df_club_members that is not present on df_stats_members due to the username being renamed or leaving the club before passing it to get_player_stats
# make inner join on username to just combine usernames that are present in both
merged_df = pd.merge(df_club_members, df_stats_members, on='username', how='inner', suffixes=('_df_club_members', '_df_stats_members'))
merged_df
```

#### Sort first by joined col then add number incrementaly on each row , will serve as primary key
 ```python
merged_df = merged_df.sort_values(by="joined")
merged_df["primary_key"] = range(1, len(merged_df) + 1)
#make primary key the first col
column_order = ['primary_key'] + [col for col in merged_df.columns if col != 'primary_key']
merged_df = merged_df[column_order]
merged_df
```

#### Derive 3 tables from this merge data frame
 ```python
# 1st: table with primary key and username only
df_users = merged_df[['primary_key', 'username']]
df_users

# 2nd: table with primary key and joined
df_joined_members = merged_df[['primary_key', 'joined']]
df_joined_members

# 3rd: table with primary key and the stats (rapid_rating, tactics_rating)
df_stats_of_members = merged_df[['primary_key', 'rapid_rating', 'tactics_rating']]
df_stats_of_members

```

#### Save as CSV files
 ```python
df_users.to_csv(r"Portfolio_DataEngineering/df_users.csv", index=False)
df_joined_members.to_csv(r"Portfolio_DataEngineering/df_joined_members.csv", index=False)
df_stats_of_members.to_csv(r"Portfolio_DataEngineering/df_stats_of_members.csv", index=False)
```
#### Next step is import in sql server





















