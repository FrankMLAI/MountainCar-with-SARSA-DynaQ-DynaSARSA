Solving OpenAI Gym's Mountain Car (https://www.gymlibrary.dev/environments/classic_control/mountain_car/) using SARSA, Dyna-Q, and Dyna-SARSA algorithms.
The Dyna-SARSA implementation is based on the "Autonomous Driving Path Planning based on Sarsa-Dyna Algorithm" whitepaper (http://fucos.or.kr/journal/APJCRI/Articles/v6n7/6.pdf).

## SARSA

As the first algorithm, this SARSA implementation establishes the groundwork for future algorithms to be discussed. As a result, several pieces of code are reused in future implementations. 
One key piece of the code is the usage of discretization in order to turn the continuous space into 400 (20 by 20) discretized buckets, enabling step-based analysis. 
A Q-Table is initialized as (discrete buckets, discrete buckets, actions) or (20,20,3) folowed by a SARSA update process.
At the end of the episodes, SARSA reached an average score of -142 from the baseline of -200, with maximums near -114. The graph shows some interesting occurrences. 
In the 8000s range, the max reward drastically drops off but the minimum reward increases significantly in proportion. 
As a result, the average value still stays the same. This behavior definitely matches the literature’s description of SARSA being a more conservative algorithm. 
Around 17000 episodes, the min value drops off again, along with a large return in max value, and overall increase in avg value from -155 to -140.

## Dyna-Q

The first step was to modify the SARSA code into one that performed Q-learning instead, thus requiring me to modify the “Setting current and future values for state and action” section of SARSA into the Q-Learning step.
The biggest challenge behind creating the Dyna-Q was to figure out how to set up model matrices that were not part of an object and could handle the state input, which was a tuple rather than an integer, making it unique from most Dyna-Q implementations. 
I created “flattening” and “unflattening” functions in order to freely transform state expressions from a tuple for interfacing with the Q_Table into an integer for interfacing with the Model matrices, which store state as an integer. 
Another challenge would be to handle the initialization of the Tables when the argmax functions would naturally select default un-updated values as the best choice until all values are filled in (since all rewards are -1). 
The usage of the model’s tables in conjunction with the Q-tables magnifies the issue of default matrices. I chose to add a flag to delay the addition of the Dyna operation until a single successful trial was completed (single instance of winning). 
This allows us to cut down on some of the downtime in the initialization process. At the end of the episodes at N=5, Dyna-Q reached an average score of -196.4 from the baseline of -200, with maximums near -190. 
While the shape somewhat resembles that of the graph of SARSA, its rewards are off by 50 reward points. I was really surprised of how terribly it performed despite the fact that most sources will describe it as superior to SARSA and Q-Learning.
However, upon repeating it with N=10, I was quite pleased to see that it’s average score had improved from -196 to -155, which is only slightly worse than SARSA, however after 10000 episodes, it had consistently high max values way above those from SARSA. 
Curious of whether I could continue improving the results at this rate, I attempted it again with N=15, but unfortunately the results were similar to those from N=5. 
It seems to me that N=10 is the closest value to the optimal value for the number of planning steps as performance tends to decrease as you either increase or decrease it.
Nevertheless, it should be recognized that the Dyna model can at times be “wrong” when it learns a suboptimal policy and magnifies the incorrectness in the planning process. 
In that sense, the discrete bucketing of 400 buckets rather than a full continuous space could yield weaker results than those that are formed from a more accurate expression of the real environment.

## Dyna-SARSA
While the "Autonomous Driving Path Planning based on Sarsa-Dyna Algorithm" was originally applied to the issue of urban traffic route planning, I saw that I could adapt it to MountainCar.
Similar to with Dyna-Q, Dyna-Sarsa also underperforms at N=5, finishing with an average near -195, which was by far the worst. However, when N is raised to 10, it shows an improvement over Dyna-Q.
It hits an average of -140 after only ~9000 episodes, which was not even accomplished by the original SARSA, which was only -155 during that episode range, only reaching -140 around 17000 episodes. 
While it managed to converge at -140 very early on, in the last few episodes it’s mins, maxes, and averages all suddenly drop.
Perhaps this drop is analogous to the drop at 8000 episodes in the SARSA graph, albeit extended across more episodes due to the effects of the difficulty of correcting a suboptimal model. 

## Conclusions
All things considered, my results show that overall, Dyna-SARSA has similar performance to SARSA with faster convergence, and both SARSA algorithms outperform Dyna-Q. 
This does not fully match the projected performance I initially predicted but there may be a wide variety of factors that contribute to the discrepancy.
My discretization steps may have oversimplified the complexity of the continuous observation space, leading to suboptimal model planning in Dyna-Q steps. 
Furthermore, the large impact of picking an optimal Dyna “N” value is another thing that was observed. Even though it performs fairly well at N=10, I believe there may potentially be even better values that will solidify Dyna-SARSA’s upgrade over the original SARSA. 
Adding a more fine-tuned epsilon modifier could also yield some improvements as the existing one is somewhat basic. Another aspect to be looked upon is adding some more sophisticated logic towards activating the Dyna flag. 
Currently, it gets activated upon the first successful trial, but there is no guarantee that Q-table or Model transition matrices are sufficiently filled prior to activation. Essentially, it is a matter of timing the introduction of the Dyna model. 
Finally, MountainCar may have not been the best game to showcase the impacts of the aforementioned algorithms as there are many other games with reward systems that are more compatible with tabular algorithms (Frozenlake, etc), or games with a greater focus on obstacle avoidance to maximize the benefits of SARSA based algorithms.
Nevertheless, the challenge of adapting them to this game was ultimately a very rewarding learning experience and I am proud to have implemented an algorithm as unique as Dyna-SARSA. It really shows me the potential of combining and modifying existing algorithms to develop improved solutions within reinforcement learning.
