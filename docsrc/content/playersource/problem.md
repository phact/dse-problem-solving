---
title: Player Scores (Problem)
menu:
  main:
      parent: Player Scores
      identifier: playerscoresproblem
      weight: 201
---

## Problem Definition

A customer wants to hold the scores of all players within their catalog of games. Theses are the requirements that we know at the moment.

1. There are currently 10 games but this will grow to 20 in 2 years. 
2. The number of players per game can range of a few thousand to 20-30 million.
3. Users can see their progress over time.
4. Players can create groups so that they can play against their friends and colleges. They can see how they can see how they are doing compared to their friends.

>E.g. 
>- High Score 131
>- Current Position - 1535th  
>- Current Position in Group 'My friends' - 3rd 
>- Current Position in Group 'DataStax Gamers' - 6th 
5. Players can look at all the scores within a group. 

The solution requires the data model and queries that will provide the above functionality.

There is a need to understand the compaction strategies and how they will affect resources. 
