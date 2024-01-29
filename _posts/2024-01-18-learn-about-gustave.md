---
layout: post
title: How Gustave works
subtitle: Strategy and details 
author: Van David LE
---

{: .box-note}
Gustave is an ev3 mindstorm robot designed during the OS course. It's purpose is to be put in an arena against another robot. Each robot needs to catch the flag situated in the other part of the arena. 

## Architecture of Gustave

### Four motors are used and 2 sensors

2 motors are used for the 2 front wheels. \
1 motor is used to activate the claw to grab the flag. \
The last motor is used to activate the trap at the back of the robot.

### Annotated pictures of Gustave

Front                                           |  Back
:----------------------------------------------:|:----------------------------------------------:
<img src="../assets/img/Front.png" width="450"> | <img src="../assets/img/Back.png" width="450">

Side                                            |  Top
:----------------------------------------------:|:----------------------------------------------:
<img src="../assets/img/Side.png" width="450">  | <img src="../assets/img/Top.png" width="450">


## Strategy to win the game

### Rules of the game

The game is taking place in a rectangular arena with a circular obstacle in the middle. The two robots must start in a square on each side. The flags are placed behind the starting point of the robots. 

![Arena](../assets/img/Arena.png)

### Strategy

Gustave will start by reach the side of the arena in order to avoid the central obstacle.

![Initial](../assets/img/Initial.png)

Then, Gustave will move forward and scan everything in front of him.
If there is nothing, he will continue to move forward until he reaches the other side of the arena.

![NoOpp](../assets/img/NoOpp.png)

If he meets an obstacle (the other robot) at some point, he will begin to rotate to avoid the obstacle from the other side of the arena. Then, he will move forward until he reaches the back of the arena.

![Opp](../assets/img/Opp.png)

Once on the opponent's side, he will come back to the middle of the arena and move in front of the flag and proceed to catch the flag.

After that, he will will come back following the same opponent detecting process than at the beginning.

![Back](../assets/img/Backtrack.png)
