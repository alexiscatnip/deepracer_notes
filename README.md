# deepracer_notes: To document deepracer learnings...
## DISCLAIMER I have only 2 weeks of deepracer experience
###

**TODO: deal with the formatting. markdown.**
**TODO: link to aws video**


# Deepracer - Getting started
## 1) Watch AWS quick start  video
It will teach you:
- How to operate the AWS online deepracer console/ training stack
- How to train your first deepracer model 
  - (I highly suggest the reward function that rewards staying close to the center line. It trained the fastest for me)

## 2) Set up Local training 
If you wish to train a model for competition, then you should  get a local training setup
  - Local training good for: 
    - Money. Online training is expensive
    - I you have GPU lying around (> 8gb recommended according )
  - Why not to use local traiing:
    - U R Rich 
1) Hardware: 
   1) CPU: You probably need a haswell+ era CPU. The author is using i5-4670, and it is able to handle the simulation environment at about 0.5 the rate of real-time
   2) RAM: I have 16GB of ram. 
   3) pc@pc-All-Series:~$ free -h
                    total        used        free      shared  buff/cache   available
        Mem:            15G        5.5G        1.8G        487M        8.2G        9.5G
        Swap:          2.0G        512K        2.0G
2) Software:
   1) I used mattcamp's training stack: https://github.com/mattcamp/deepracer-local

3) OOM errors from tensorflow
   1) just reduce batch size to 32 or 16.
   2) not sure there is any compensation to make for this.
   3) or get  gpu with bigger memeory

# How to train for comp? (steps to take, reward functions)
1. First priority is to have the vehicle complete the lap consisitently. 
   - Suggest to train on "stay on the center line" reward function (default AWS).
   - Do this until: the vehicle can  CONSISTENTLY race the track, without going off-course during evaluation.
   - Instead of using the center line, you can use waypoints. This in theory should help your car learn to cut corners! But in my experience the vehicle often goes off-course during training, which resets it to the center line, which can be far from the waypoint. This messes up the rewward algorithm that is based on waypoints, i think.
3. Next priority is upping the speed as you like it. Be ambitious or not, whatever. Virtual track is slidey. 
   - Increase speed in small increments in the action space.
   - Train it in the new speed until it learns its new speed. Ensure it can complete the track.
   - You can consider changing the angles depending on the track. But the default 30-20-10-0 degrees action space is fine.
4. Finally, optimise it for speed. 
   - Add "speed" parameter into the reward function. Teach the bot to go fast fast.
   - Recommended to use progress as a factor of your reward function. This helps prevent the bot from flying off the track on bends due to "reward-function-favoring-pure-speed". 
   - Note that "speed" parameter is just current speed. It is not average speed. Increasing speed factor in the reward function can cause dumb behavior.

### things that I still cant get around:
   - vehicle not want to take high speed (4m/s straights)
   - reward fuinction often rewards sub-optimal lap time (24second completion has higher total reward than 22s completion) despite the progress variable
   - not sure how to best optimise action space (need manual work. how do you know what is the optimal action set?)
   - hyperparameters
   - reliability at higher speeds (can lap 2/5 times)
   - how to tokyo drift (lol)
   - how to reward aggressive curve-cutting without going totally off-track
   - How to achieve smooth turns (similar to previous)

# How does the deepracer "brain" (machine learning model) work?
## Diagram explanation:

### What happens during evaluation? (aka running the model)
```
INPUT (camera image/state)--->   MODEL (like a function) ------> OUTPUT (ACTION SPACE, aka MOVEMENT)

> The model simply tells the car what actions to take, when presented with the current camera image. 
> EG: If camera sees an image of a LEFT TURN, the action to take should be "turn left 20deg". 
      Repeat this in a loop, and the car magically drives itself around the track.

```
### What happens during simulation? (aka training the model)
```
INPUT (camera image/state)--->   MODEL  (like a function) ------> OUTPUT (ACTION SPACE, aka MOVEMENT)
                                   /\
                                  /  \     
                                 /_  _\     
                                   ||
                                   ||
                                reward (a positive number, more reward means bigger number)

> The key difference is that there is a REWARD thingy. 
> This REWARD indicates the "good-ness" of the INPUT-OUTPUT pair (the CAMERA_IMAGE - ACTION_SPACE pair) 
> EG: 
      > Stupid action
      Camera image: LEFT TURN
      action TAKEN: RIGHT TURN
      REWARD = LOW REWARD (for stupid action)

      > Smart action
      Camera image: LEFT TURN
      action TAKEN: LEFT TURN
      REWARD = HIGH REWARD (for smart action)

```
## To summarise, a good model is one that has the **appropriate "state-action" pairs** that give you desired behavior (able to drive on track fast).
## The sole responsibility of the **reward function** is to tell the model-in-training, what is the **desirable action** to take for the present **state**.


# Lol dont read this 
1) (My understanding!!! might not be accurate) The idea of training is to establish and tune the **model** by gathering date about **action-state pairs** through simulation. Example:
```
|       |       |
|       |       |
|       |       |
|       |       |
|       |      /\
|       |     /  \ good job baby cat
|       |     \__/
|       |       |
|       |       |
|       |       |
|       |       |
|       |       |
```
   - 
   - Situation: The car is driving along the RIGHT SIDE of the road, dangerously close to coming off the road.
   - Action space: The action space is the "output" of the 
   - The car can TURN LEFT back to the road, or turn RIGHT off the road into oblivion.
   - The reward function gives a reward if the car is on the road, and no reward if it is off the road.
   - The car stores its current **state** (car-camera data, depicting it being really close to the road-side)
   - The car also stores its current **action**, either "turn left "
1) I suggest training your deepracer 