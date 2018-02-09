---
title: Player Scores (Solution)
weight: 20
menu:
  main:
      parent: Player Scores
      identifier: playerscoressolution
      weight: 202
---

Requirement 4 is the only one that makes this interesting.

## Data Modeling Considerations

The usual way we would think about this is having a partition with all the scores and then having the cluster key be the score and maybe unique player code.

In this case, this won’t work as the clustering key will be continuously changing and the partitions could also be incredibly big. Could we break up the partitions to bucket it in some way?

This may well work in some scenarios but with the clustering column changing so frequently it may cause some problems and it makes the application code a lot more complex.

Since the access patterns vary (requirement 4), we may not want to limit our solution to CQL.

## DSE Features?

Spark may well be able to process the stream in realtime and try to have an up to date cache of scores for each game. It could then be queried by the job server or similar. Again this makes the solution very complex because all the processing become part of the spark engine and its cache.

So what about a graph? What would we be using a graph for exactly ? The players groups ? The players games ?

What we really want to do is search how many people that have played a certain game have a score over a certain score.

## Leveraging DSE Search

So - doing this in SQL would be pretty simple.

Get all the number of people in game 1 with a score higher than mine

    Select count(id) from high_scores where gameid=1 and score>500;

Get all the number of people in game 1 with a score higher than mine and in the group datastaxers

    Select count(id) from high_scores where gameid=1 and groups = ‘Datastaxers’ and score>500;

So if we take the same principle, we can do the exact same with DSE Search.

Get all the number of people in game 1 with a score higher than mine

    select count(*) from datastax.high_scores_groups where solr_query = '{"q":"*:*", "fq":"gameid:1 AND highscore:[500 TO *]"}';

Get all the number of people in game 1 with a score higher than mine and in the group datastaxers
    
     select count(*) from datastax.high_scores_groups where solr_query = '{"q":"groups:datastaxers AND highscore:500", "fq":"gameid:1"}';

For this implementation we will use DSE Search. Each game with have rows for the highscore for each player the groups that the user is part of for that game. 

## Player Game history 

This is straightforward dynamic table with playerid, gameid being the partition key and time being our clustering column.

```
create KEYSPACE if not exists datastax WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

create table if not exists datastax.player_game_history (
playerid text,
gameid text,
scoredate timestamp,
highscore int,
PRIMARY KEY ((playerid, gameid), scoredate)
) with clustering order by (scoredate desc);
```

## Scores with Groups

For each game, the player can be a part of 0 to many groups. So we can model the groups as set of groups and kept with the  high scores. For clarity I am using group names but in reality these would be group ids.   
```
create KEYSPACE if not exists datastax WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

create table if not exists datastax.high_scores_groups (
gameid text,
playerid text,
highscore int,
scoredate timestamp,
groups set<text>,
PRIMARY KEY (gameid, playerid)
);
```

Create Core
```
create search index on datastax.high_scores_groups;
```

Insert data for groups and scores.
```
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','1', {});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','2', {});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','3', {'Friends', 'DataStaxers'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','4', {'Friends', 'DataStaxers'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','5', {'Friends', 'DataStaxers'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','6', {'Friends', 'DataStaxers'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','7', {'Friends'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('1','8', {'Friends'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','1', {});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','2', {});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','3', {'Friends'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','4', {'Friends'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','5', {'Friends'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','6', {'Friends'});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','7', {});
insert into datastax.high_scores_groups (gameid,playerid, groups) values ('2','8', {});



insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','1', 12313);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','2', 1233);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','3', 343);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','4', 12533);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','5', 7547);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','6', 236);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','7', 757);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('1','8', 26);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','1', 13);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','2', 13);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','3', 141);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','4', 114);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','5', 89);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','6', 2);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','7', 722);
insert into datastax.high_scores_groups (gameid,playerid, highscore) values ('2','8', 103);
```

To run the queries needed 

First find the current highscore and groups for the game and player - this would probably be while starting the game. 
```
select highscore, groups from datastax.high_scores_groups where gameid = '1' and playerid = '4' ;
```
At the end of the game, the user has a high score of 500, if this is greater that their last high score, we use this in the next queries. 

What is my position overall in the game ?
```
select count(*) from datastax.high_scores_groups where solr_query = '{"q":"highscore:[500 TO *]", "fq":"gameid:1"}';
```

What position am I in for all my groups ? 
```
select count(*) from datastax.high_scores_groups where solr_query = '{"q":"groups:datastaxers AND highscore:[500 TO *]", "fq":"gameid:1"}';

select count(*) from datastax.high_scores_groups where solr_query = '{"q":"groups:datastaxers AND highscore:500", "fq":"gameid:1"}';

select count(*) from datastax.high_scores_groups where solr_query = '{"q":"groups:friends AND highscore:[500 TO *]", "fq":"gameid:1"}';

select count(*) from datastax.high_scores_groups where solr_query = '{"q":"groups:friends AND highscore:500", "fq":"gameid:1"}';

```
NOTE: notice the use of the queries and filter queries to make use of the filter cache. 

Get the leaderboard for in a game for a particular group. 
```
select playerid, highscore from datastax.high_scores_groups where solr_query = '{"q":"groups:datastaxers", "fq":"gameid:1", "sort":"highscore desc"}';
```

## Compaction 

The compaction strategy for the high scores table should be Levelled Compaction due to the update and mostly read profile. We will be compacting out all old high scores and dates quite regurlarly and the table growth should be steady. 
