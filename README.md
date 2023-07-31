# ML-Agents DodgeBall Extended Battle Scenario

## Overview

The [ML-Agents](https://github.com/Unity-Technologies/ml-agents) DodgeBall environment is a third-person cooperative shooter where players try to pick up as many balls as they can, then throw them at their opponents. It comprises two game modes: Elimination and Capture the Flag. In Elimination, each group tries to eliminate all members of the other group by hitting them with balls. In Capture the Flag, players try to steal the other team’s flag and bring it back to their base. In both modes, players can hold up to four balls, and dash to dodge incoming balls and go through hedges. You can find more information about the environment at the corresponding [blog post](https://blog.unity.com/technology/ml-agents-plays-dodgeball).

In this project, we used the Elimination game-mode to explore modifying the DodgeBall environment to serve as a proxy for more realistic military simulations. We modified both the dodgeball agents' functionality and the arenas they were tested in to better approximate a high fidelity battle scenario. This document will detail the most significant changes we made and discuss the method we developed for reducing the number of training steps needed to learn an intelligent cooperative policy. 

## Installation and Play

To open this repository, you will need to install the [Unity editor version 2020.2.6 or later](https://unity3d.com/get-unity/download).

Clone the `dodgeball-env` branch of this repository by running:
```
git clone https://github.com/Unity-Technologies/ml-agents-dodgeball-env.git
```

Open the root folder in Unity. Then, navigate to `Assets/Dodgeball/Scenes/TitleScreen.unity`, open it, and hit the play button to play against pretrained agents. You can also build this scene (along with the `Elimination.unity` and `CaptureTheFlag.unity` scenes) into a game build and play from there.

## Scenes

In `Assets/Dodgeball/Scenes/`, in addition to the title screen, eight scenes are provided. They are:
* `Large_F_Obs.unity`
* `Large_F_Obs_Dense.unity`
* `XL_F_Obs.unity`
* `XL_F_Obs_Dense.unity`
* `Large_WPM_Obs.unity`
* `Large_WPM_Obs_Dense.unity`
* `XL_WPM_Obs.unity`
* `XL_WPM_Obs_Dense.unity`

Where Obs differentiates between the scenarios which include the modified observation space and those that do not. WPM stands for waypoint manual, which was the final iteration of our waypoint movement system and will be discussed in the Waypoint Movement section of this document.  Large and XL refers to the two different sizes of arena in which we tested our waypoint movement system against the original continuous implementation. F simply indicates that the scene is the final build of our project. 

### Elimination

In the elimination scenes, four players face off against another team of four. Balls are dropped throughout the stage, and players must pick up balls and throw them at opponents. If a player is hit twice by an opponent, they are "out", and sent to the penalty podium in the top-center of the stage.

![EliminationVideo](/doc_images/ShorterElimination.gif)

The original dodgeball environment includes the option for capture the flag, but we did not it during the course of this project. All results and scenes take place in the Elimination gamemode. 

## Training

ML-Agents DodgeBall was built using *ML-Agents Release 18* (Unity package 2.1.0-exp.1). We recommend the matching version of the Python trainers (Version 0.27.0) though newer trainers should work. See the [Releases Page](https://github.com/Unity-Technologies/ml-agents#releases--documentation) on the ML-Agents Github for more version information.

To train DodgeBall, in addition to downloading and opening this environment, you will need to [install the ML-Agents Python package](https://github.com/Unity-Technologies/ml-agents/blob/release_18_docs/docs/Installation.md#install-the-mlagents-python-package). Follow the [getting started guide](https://github.com/Unity-Technologies/ml-agents/blob/release_18_docs/docs/Getting-Started.md) for more information on how to use the ML-Agents trainers.

You will need to use either the `CaptureTheFlag_Training.unity` or `Elimination_Training.unity` scenes for training. Since training takes a *long* time, we recommend building these scenes into a Unity build.

A configuration YAML (`DodgeBall.yaml`) for ML-Agents is provided. You can uncomment and increase the number of environments (`num_envs`) depending on your computer's capabilities.

After tens of millions of steps (this will take many, many hours!) your agents will start to improve. As with any self-play run, you should observe your [ELO increase over time](https://github.com/Unity-Technologies/ml-agents/blob/release_18_docs/docs/Using-Tensorboard.md#self-play). Check out these videos ([Elimination](https://www.youtube.com/watch?v=Q9cIYfGA1GQ), [Capture the Flag](https://www.youtube.com/watch?v=SyxVayp01S4)) for an example of what kind of behaviors to expect at different stages of training.

### Environment Parameters

To produce the results in the blog post, we used the default environment as it is in this repo. However, we also provide [environment parameters](https://github.com/Unity-Technologies/ml-agents/blob/release_18_docs/docs/Training-ML-Agents.md#environment-parameters) to adjust reward functions and control the environment from the trainer. You may find it useful, for instance, to experiment with curriculums.

| **Parameter**              | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ball_hold_bonus`| (default = `0.0`) A reward given to an agent at every timestep for each ball it is holding.|
| `is_capture_the_flag`| Set this parameter to 1 to override the scene's game mode setting, and change it to Capture the Flag. Set to 0 for Elimination.|
| `time_bonus_scale`| (default = `1.0` for Elimination, and `0.0` for CTF) Multiplier for negative reward given for taking too long to finish the game. Set to 1.0 for a -1.0 reward if it takes the maximum number of steps to finish the match.|
| `elimination_hit_reward`| (default = `0.1`) In Elimination, a reward given to an agent when it hits an opponent with a ball.|
| `stun_time` | (default = `10.0`) In Capture the Flag, the number of seconds an agent is stunned for when it is hit by a ball.|
| `opponent_has_flag_penalty`| (default = `0.0`) In Capture the Flag, a penalty (negative reward) given to the team at every timestep if an opponent has their flag. Use a negative value here. |
| `team_has_flag_bonus`| (default = `0.0`) In Capture the Flag, a reward given to the team at every timestep if one of the team members has the opponent's flag.|
| `return_flag_bonus`| (default = `0.0`) In Capture the Flag, a reward given to the team when it returns their own flag to their base, after it has been dropped by an opponent.|
| `ctf_hit_reward`| (default = `0.02`) In Capture the Flag, a reward given to an agent when it hits an opponent with a ball.|


# Extending DodgeBall to Emulate Military Training Scenarios 

## Infinite Ammunition 
The first change that was needed to convert the original dodgeball scenario into a more military-esque scenario was infinite ammunition. We implemented a system that destroys projectiles on impact and returns them into the possession of the agent. This removes the need to go and recover balls, which does distracts from tactical movement and adds an unnecessary layer of complexity for the agents to learn. 
![](https://github.com/calebkoresh/ml-agents-dodgeball-env-ICT/blob/develop/Media/infinite_ammo_video_AdobeExpress.gif))

## 3D Terrains 
The next step was developing more realistic terrain. Battle scenarios will seldom occur on flat ground, so we imported data from the Razish Army Training Facility which allowed us to train our agents on a low-fidelity version of real-world training terrain. 
Picture
This new setup requires additional raycasts, so that the agents can detect opponents or walls that are not at the same altitude as them. This is crucial for developing intelligent policies when uneven terrain is introduced. 
![](https://github.com/calebkoresh/ml-agents-dodgeball-env-ICT/blob/develop/Media/Screenshot%20(27).png)
## Modified Observation and Action Spaces 
Some modifications were made to the agents observation and action spaces were made to better fit our needs. The observation spaces are smaller due to the removal of unnecessary observations that only apply to the Capture the Flag gamemode. Additionally, the dash action was removed as it was a bit awkward in our scenario, especially when moving along waypoints. 
## Shooting Vertically 
Another obvious additon to our scenario was the ability to shoot vertically. Opponents should be able to fire at angles other than parallel to the ground so that they can target opponents at various different altitudes. 
![](https://github.com/calebkoresh/ml-agents-dodgeball-env-ICT/blob/develop/Media/Autoshoot_vertical_gif_AdobeExpress.gif)
## Aim-assist 
We attempted to train some models which were able to choose the angle of their shots, but this drastically increases the complexity of the environment. Our solution was to implement aim-assist, which targets the opponent closest to straight ahead and then automatically fires directly at it. This removes the need for fine tuning aim and encourages learning intelligent positioning and movement over high-precision skills. This method achieved far better results, so it was used in most of our simulations and all the experiments in this repository. 
![](https://github.com/calebkoresh/ml-agents-dodgeball-env-ICT/blob/develop/Media/autoshoot_demo_gif_AdobeExpress.gif)

## Introducing Roles 
We also investigated the ability to introduce different roles within the same team. We hoped to see whether the agents could learn a more complicated strategy to cooperate and utilize each individuals strengths. This was studied using short and long range units with different capabilities. The short-range units have half the aim-assist range but twice the fire-rate. We found that the agents did in fact learn their role. Short-range units learned more aggressive policies and the long-range units tended to remain in rear. The long-range units can be distinguished by their darker color. 


https://github.com/calebkoresh/ml-agents-dodgeball-env-ICT/assets/80787784/e9222e3b-f3ae-4f1b-8e15-41123a8eb155



## Waypoint Movement 
Due to the large computational requirements of reinforcement learning, we were not able to run our simulations for the same 160 million training steps that the original project did. This fact combined with the increased complexity of our environments led us to develop a method to reduce training time. We developed a waypoint movement system which aims to reduce the complexity of our environments and reduce the frequency of reinforcement learning steps while retaining the core positional strategy. This system limits agents to walking along the waypoints we generate onto the terrain. This allows us to automate shooting and only utilize reinforcement learning for the agents' movement. This allows us to only request a decision from the learned policy at each waypoint which is translated to the direction to travel to the next waypoint. This increases the time between decisions by over 700%. We also developed a system to automatically generate these waypoints so the system can be quickly implemented on any unity terrain.
Demo Video of agents on waypoints 

## Tests 
We tested the waypoint movement system against the original continuous version in four different scenarios, including two sizes and two obstacle densities. 
Arena Pics 
Result Videos 


https://github.com/calebkoresh/ml-agents-dodgeball-env-ICT/assets/80787784/d7ea3ae4-781f-4ec5-a383-02830fa262b1



Additionally, we tested the two systems directly against each other by converting the waypoint-based decisions back into continuous movement with eight possible directions. 
Movement gif
Result Videos 

## Results 
Data 



