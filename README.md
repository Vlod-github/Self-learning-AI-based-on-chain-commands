# Self-learning AI based on a chain of commands
The aim of the work is to create an AI with unusual "natural" behavior, the learning process of which automatically adjusted to changes in the external environment with minimal developer participation.

In this repository, an example of a self-learning (reinforcement learning) AI (hereinafter referred to as an agent) is considered. The AI architecture is a chain of commands, and the genetic algorithm (GA) will select these commands. Next, we will consider: the environment in which the testing (game) will be conducted, the AI command system, the GA operation and the agent evaluation function.

https://user-images.githubusercontent.com/103655830/177856051-8b0ba07d-51d3-412c-9937-8441edeb3d64.mp4

https://user-images.githubusercontent.com/103655830/177856170-aceaa649-9737-4f95-a9c3-4efd4994913f.mp4
## Environment
Testing takes place in the warcraft 3 1.26a game with an old license agreement that still respects intellectual property. We will train agents to fight in a mini-game similar to the Hunger Games, where the goal is to survive and kill opponents.

The medium is a shrinking circle. If an agent goes outside the circle, he quickly loses health. In the center of the circle are aggressive creatures with simple intelligence that look like zombies. They attack agents as soon as an agent appears in sight. When the circle reaches the minimum size, then everything starts over.

![area](https://user-images.githubusercontent.com/103655830/177856388-cfeaf0f4-3418-46bd-b2e2-959dde52e070.png)
## Agent
An agent is a game object that has the amount of life, the amount of attack, the speed of movement and two abilities: invisibility for a while and stun with damage in a straight line in the form of spikes. The agent selects an action every 0.33 seconds.
## Command System
The agent's command system is an integer array with a length of 64. Each cell contains one command. The maximum number of commands an agent can execute at a time is 16. Execution always starts from the first cell of the array. Commands are divided into 2 types: actions and conditions.

![chain](https://user-images.githubusercontent.com/103655830/177857601-22e13dbd-713d-4c48-b9aa-d29edd944edb.png)

![command](https://user-images.githubusercontent.com/103655830/177856711-60b50514-3018-4008-94d9-bad33463a20b.png)

An action command can have a positive and negative response (if the action cannot be performed). If the answer is positive, then the agent's work is terminated. If the answer is negative, the pointer moves to the next cell of the command array.

![action](https://user-images.githubusercontent.com/103655830/177856738-fb3900c1-40cc-42f1-b69a-0e2d8f333397.png)

The condition command also has a positive and different response. If the answer is positive, the pointer moves to the next cell of the command array. If the answer is negative, then there is a transition to another cell from the array of commands.

![condition](https://user-images.githubusercontent.com/103655830/177856770-8643d115-98b2-43a5-af74-cc93f5be06e5.png)
## Training
At the same time, 12 agents participate during the training. After each round, agents are evaluated and their usefulness is adjusted.

The training uses a genetic algorithm that works as follows:
after the end of round, we evaluate the agents and replace the 4 worst ones. One new agent appears by mutation operation. The second one appears by random generation, and the third and the fourth by a one-point crossover operation.

![iter](https://user-images.githubusercontent.com/103655830/177856847-93e94447-9843-4e62-9d10-ce4cd0049bb7.png)

I note that during the selection of candidates on the basis of which new agents will be created, we take the best individuals with a higher probability, and the worst ones with a lower probability. This approach shows greater efficiency than if you choose random agents.
<details>
  <summary>Additional</summary>
  
Due to the fact that all 12 agents are evaluated after each round, we get an error estimate because the agents end up in different initial conditions. In return, we get an acceleration of 12 times. As you understand, with 100 agents, we will get a 100-fold acceleration, because ideally we need to test by placing each agent in the same environment. This error can significantly affect our assessment, which is why the GA will miss effective agents. To combat this negative effect, we use 2 methods:
1) The first method is that each subsequent evaluation does not replace the previous one, but corrects:
`value[i] = value[i-1] + (value[i-1]-cur_value)*(1/10)`, where `value[i]` is the adjusted score, `value[i-1]` is the score for the previous round, `cur_value` is the score based on the results of one current round, and 10 is the coefficient. As the coefficient increases, the estimate will change more slowly, but it's averaging will occur more accurately. With small values of the coefficient (1, 2), random bad conditions can give a bad estimate for a good agent, then the final estimate will change too abruptly and the agent will be disposed of (lost forever).

2) The second method is to dispose of bad agents not every round, but every N rounds. We chose N = 2. This improves the accuracy of the estimate, because it reduces the random error, but slows down our algorithm by 2 times. But it's still much faster than if we were doing a fair 12 rounds for a single GA iteration.
</details>

## Evaluation function
This is the algorithm by which we will evaluate the agents. We use the following formula:
```
r = (job_kill + job_count)*0.5
job = (job_survived*r + r)*0.5
```
where `job_survived` is the value that is in state 1 if the agent is alive at the end of the round, otherwise 0, `job_kill` is the normalized [0, 1] number of kills that the agent has committed in this round, and `job_count = 1. - live_agent/max_agent` [0, 1]  (`max_agent` is the total number of agents, and `live_agent` is the number of live agents at the time of the agent's death or at the end of the round)

This formula was constructed through experimentation and intuition. In reality, the agent's estimate is a surrealistic number or probabilistic estimate, so our estimate is a reflection of some real estimate. Of the specific recommendations, the only one is that it should be continuous with respect to the tasks you put in front of the agent. That is, higher-quality agents should have a higher score, and lower-quality ones should have a lower score, and the training function should be able to notice small changes in the behavior of agents.

## Results and conclusions
After the training, we put the AI we liked into a test mini-game that you can run on your computer and test them yourself. For those who do not have such an opportunity, we attach a video:

https://user-images.githubusercontent.com/103655830/177857789-4f001fb6-55d6-4e75-8c95-590d9c4c5323.mp4

https://user-images.githubusercontent.com/103655830/177857835-accb5390-8fb8-42d4-ac77-305f6e15551f.mp4

It turned out pretty well. The main advantage of this approach is that we have absolutely no idea about the rules that the external environment sets and how the agent interacts with it. To work, it is enough just to be able to evaluate the effectiveness of an agent using some function.

You can develop the algorithm in different directions, for example, expand the command system, which will allow you to obtain completely new behavior models, use adaptive hyperparameters for GA, parallelize by running several simulations with subsequent integration. Good luck!

## Launch and modification
In order to launch .w3x maps you need any warcraft 3 version 1.26+

In order to view the code it is enough to open the map in the World Editor, which is in the folder with warcraft 3, but JassNewGenPack is recommended, because it has TESH built in, which has a jass syntax highlighting

In order to modify and save these maps you need a JassNewGenPack with cjass support, such as [this](https://xgm.guru/p/wc3/jassnewgenpack-r)