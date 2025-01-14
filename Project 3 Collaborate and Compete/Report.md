# P3 - Collaboration and Competition

<p align="center">
<img src ="https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Output.gif">
<br>
<b> Solved Environment <b>
</p>

This report details the algorithm and implementation for the Collaboration and Competition project of the Deep Reinforcement Learning Nanodegree on Udacity.

The environment was solved using the DDPG algorithm (suitable for environments with continuous action spaces) which involves the use of four neural networks, two each for both actor and critic. DDPG was used here instead of the more reliable MADDPG, as an initial experiment to see the effect of non-stationary environments as detailed in the MADDPG paper. But turns out that the DDPG agent with multiple agents used for experience collection as done in the continous control project performs fairly well. On trying to read about it turns out, that it was not a fluke, but an observation of many students in the course, where some resources even went on to attribute it to the symmetry of the environment which seemed like a logical assertion. 

Architecture of Neural Networks :
  1. Actor : The actor target and local networks had four layers, out of which two were hidden with 64 and 64 units respectively. 
  2. Critic : The critic target and local networks also had four layers, again out of which two were hidden with 64 and 64 units respectively. 

Exponential Linear Unit was used instead of ReLU or Leaky ReLU based on the both personal experience and advise from other students on the student hub, for both actor and critic networks. In this project, due to the good results given by ELU in the previous projects, ELU was the activation function which I started with and it gave good results from the very first go. Hence, for this specific project ReLU and leaky ReLU were not played with.  The function used as activation in output layer of actor network was tanh owing to the range of action space being -1 to 1, whereas the activation function used in output layer of critic network was linear acting as value function values meant to act as critic for the agent. 

![Actor network Code](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/ActorNetwork.png)
![Critic network Code](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/CriticNetwork.png)

The code for the agent system (generalising for >1 agents, distributed learning) comprises of agent class implementation that involves initialisation of the four networks, replay buffer, and OUNoise. The target and local networks for both actor and critic are initialised with same weights (Indicated to give good performance on Student Hub). The class also includes act method that gives the array of actions for the different states observed by different agents of the environment by passing the states through the actor network, along with a hint of exploration included using epsilon based chosing (if a random number generated using numpy is greater than than the current value of epsilon that noise is added to the action vector else it isn't. As the value of epsilon decays over time, the less is the probability of exploration, and that is exactly is the basic premise behind exploration and exploitation. This method is used to gain experience on the basis of current policy enacted by the actor network. We also have the step function which stores the current sets of states and actions along with their rewards for all 20 agents one by one in the replay buffer. This increases the amount of exploration done by the agents improving the time taken for the training of the model to happen. This method is decoupled from the update part unlike as implemented in the ddpg pendulmn notebook provided by Udacity. To be able to update without needing to explore, another method was defined. Next is the reset function which resets the noise object employed by the agent system per episode. Then we have the learn method which involves, obtaining the next actions on the basis of the next states in the sampled experiences from the replay buffer, from the actor network. Using these actions, Q_targets are estimated and are compared with Q_expected for the current set of states and actions to calculate the loss function using the mean square error function on which the optimisation of the critic is done. Next the actor network is optimised using the predictions of actions for the current states, and passing them through the critic network. That is followed by soft updates of both target networks.

![Agent Code 1](https://github.com/championballer/RL/raw/master/P2_Continuous_Actions/Photos/Agent1.png)
![Agent Code 2](https://github.com/championballer/RL/raw/master/P2_Continuous_Actions/Photos/Agent2.png)
![Agent Code 3](https://github.com/championballer/RL/raw/master/P2_Continuous_Actions/Photos/Agent3.png)
![Agent Code 4](https://github.com/championballer/RL/raw/master/P2_Continuous_Actions/Photos/Agent4.png)


The OUNoise (Added to add exploration in continuous action based environments) class is same as that employed in the ddpg pendlumn with the difference that it is adapted for 2 different agents by passing the num agents argument in the size argument of the class, which allows to sample the required dimensions from the standard normal for noise generation, plus standard normal distribution based noise is added (Student Hub tip). This is followed by the replay buffer class which is exactly same as that of the one in ddpg pendulmn and the continous control project. 

The parameters for the agent system is as follows :
```
Buffer size (Replay buffer size) : 100000
Batch size (Mini batch size): 128
Gamma (Discount rate) : 0.99
Tau (Soft update influence rate) : 0.2 (Taken from a udacity discussion)
Learning rate for both actor and critic networks : 0.001 (Taken from a udacity discussion)
Weight decay for critic : 0

Starting value of epsilon : 1
Decay rate of epsilon : 0.0001
Min value of epsilon : 0.05

Optimiser used : Adam optimiser
Loss function : Mean square error function

1 learning update per step for both networks 
```
![Parameters](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/Parameters1.png)

The ipython notebook majorly consists of loading depedencies and initiating the environment and agent. After that we implement the DDPG function with default parameters of 2000 episodes, with maximum of 1000 steps in an episodes, and toggle off (to update in alternate steps if on, implemented to update 10 times in 20 steps as indicated by udacity's implementation of the environment for continuous control project). The function also uses the update_num variable which decides how many updates should happen per step given that a lot of experience is being collected per step. One deque of 100 size and one array named scores_deque and scores respectively keep track of mean scores of all agents for the last 100 and all episodes respectively. Once the mean score of the last 100 episodes crosses +0.5, the function breaks out of the main loop to conclude that the environment is solved and project is completed. After that the plot for score for all episodes is made. 

![MainCode1](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/MainCode1.png)
![MainCode2](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/MainCode2.png)
![MainCode3](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/MainCode3.png)
In our case the final process took 692 episodes to solve, and the plot for the same is as follows (also present in the implementation).

![Rewards plot](https://github.com/championballer/RL/raw/master/P3_Collab_Compete/Images/PlotFinal.png)

## Future Possible Improvements:

1. As described by Udacity in the benchmark implementation, the agent would be implemented using MADDPG algorithm to increase stability and efficiency of the model as well as to experimentally obeserve on what parameters MADDPG outplays DDPG for this environment.
2. Further fine tuning of the hyperparameters for exploration-exploitation dilemma along with other agent parameters will be tried to further optimise the agent's performance in the environment.
3. Multiple other architectures for both the actor and critic target and local networks will be tried, to increase or decrease training time along with checking up on stability of learning. 
