# Baby steps towards a Halite Reinforcement Learning agent :robot:

This kernel contains the first steps I have taken in creating a reinforcement learning agent for Halite IV playground edition : https://www.kaggle.com/c/halite-iv-playground-edition

Being a beginner in reinforcement learning, I have worked my way up in baby steps using the tf-agents package. 

At this point, here are the steps that I have taken : 
1. Create an environment and teach an agent to control a single ship, with no opponents. Reward is collected halite. Agent learns to maximise collected halite, by mining the highest containing halite cells of the board. 
2. Add a single opponent ship, moving randomly without collecting any halite. Reward is still collected halite but with a high penalty if the agent ship is destroyed. Agent learns to maximise collected halite while avoiding the opponent ship. 
3. Use the agent trained at step 2 as an opponent to train a second agent. Reward is now the difference in collected halite between the agent and his opponent. Agent must learn to optimise his halite collection better than his opponent. 

All these steps are applied on a simplified version of the game : The board is 7x7 and the game lasts 20 turns (instead of 21x21 and 400 turns). 

The main difficulties I still have to overcome to train an agent on the full game are : 
1. How can you enable the agent to control multiple ships ? You can see that in this code the agent observation is defined as relative to the ship the agent must control. My goal is to call the agent as many times as there are ships, giving it each time the specific observation for this ship. For this, the order in which the ships must be submitted to the agent needs to be defined. Adapting the reward is also a challenge. 
2. How to implement "league training" ? Having the agent train against multiple versions of himself in a virtual league in order to make sure the agent does not overspecialize in defeating a particular strategy and still manages to defeat earlier versions of himself. 

Unfortunately the Kaggle kernel does not yet include tf-agents, so it is not possible to submit a bot to the competition using this package. I am hoping they will add it for future Halite competitions ! 

Any feedback or comment will be greatly appreciated ! Like I said, I am beginning in Reinforcement Learning so there are probably loads of ways this code can be improved. 


# Code description
The code is based on the tf-agents DQN tutorial for the cartpole gym environment : https://www.tensorflow.org/agents/tutorials/1_dqn_tutorial?hl=en.

## 1.Environment 
To adapt the tf-agents cartpole tutorial to Halite, the main difficulty was to create a compatible environment. In the tutorial, the cartpole gym environment already provides everything needed for tf-agents to interact with an agent. For Halite, a custom environment compatible with tf-agents must be created. 

I created this environment by following a tic tac toc tutorial (https://towardsdatascience.com/creating-a-custom-environment-for-tensorflow-agent-tic-tac-toe-example-b66902f73059). Basically, we need to create a python environment class containing the following functions : 
* init : Create a board instance and initialize internal variables. 
* _step : Apply the action chosen by the agent to the board and calculate reward. Return observation and reward in a time_step. 
* _reset : Reset board at the end of a game. 

I also added the following functions to the environment (I guess they could be placed elsewhere, in a helper class for example):
* get_observation : Get agent observation from the current board state. 
* set_opponent_behaviour and opponent_agent : Define opponent behaviour. 

## 2.Observation
The observation is defined relative to the current ship the agent must control. For now, the agent has two "radars", one containing the surrounding halite, the other one containing the ally and enemy ships. For ships, I have distinguished ally ships (we don't want to collide with those), enemy ships with more cargo (we want to collide with those) and enemy ships with less cargo (we absolutely don't want to collide with those). 

## 3.Reward 
The reward is the difference in halite collected between the agent ship and the opponent ship. If the opponent's behaviour is defined as randomly moving, then its collected halite will always be 0. 

## 4.Neural network 
The agent neural network is a Conv2D layer followed by two Dense layers. Conv2D is used as a feature extractor. 

I have not tested any alternative architecture. This simple neural network gave good results on the simplified version of the game. 