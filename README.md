# RoboND Project (March 2018)
## Project: Search and Sample Return
### Divya Patel (OptimomEngineer)
---
This course project allowed me to gain experience with perception, decision making and action associated with autonomous robots. Similar to the NASA Sample search and pickup, I learned to program a robot to navigate autonomously in simulation and create a map of its environment while searching for rock samples

**The goals / steps of this project are the following:**  

1) **Training / Calibration**  
Add functions to detect obstacles and samples of interest - in our case, rocks. Perform appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.
Create a movie with moviepy to show completed processing.

2) **Autonomous Navigation / Mapping**

Perform appropriate image processing functions to create a map and update a class associated with the robot instance. Complete decision tree script with conditional statements that take into consideration the outputs of the image processing in deciding how to issue throttle, brake and steering commands. Iterate the perception and decision function until the rover does a reasonable (need to define metric) job of navigating and mapping. 

[//]: # (Image References)

[image1]: ./misc/initial_samples.png
[image2]: ./misc/Pers_transform.png
[image3]: ./misc/thresh1.png 
[image4]: ./misc/thresh2.png
[image5]: ./misc/arrow.png
[image6]: ./misc/simulator_settings1.png
[image7]: ./misc/simulator_settings3.png



### To summarize the project.
I warped each camera image obtained from the robot camera to get some information of what the robot was looking at and applied a threshold to that image (like a mask) in order to determine navigable terrain vs non-navigable terrain (and find rocks!). After that, I worked on the decision tree for the robot to act on what information the current image was providing.  I am still currently developing the decision tree further in order to retrieve all the rock samples rather than just the rock the robot encounters when in a corner. Currently, the robot slows down to pick up the sample, but the sample moves out of perception and so the robot no longer sees the rock and then continues on its way to the next rock.  I'm currently developing a Markov chain to improve the robots ability to see a sample rock and approach at a nominal velocity in order to slow down enough to stop and pick up the sample. I also mapped the environment in real time as the robot processed each image in the camera.  I was able to map all the rock samples and build a mapped navigable terrain environment with over 68% fidelity.


I first started the Rover simulator and recorded a stream of data by moving the robot through the terrain.
I ran the functions provided in the notebook both on test images and also on a random image that I pulled from the recorded data.
Here is a sample of the image I pulled from recorded data along with the two test images I used.
![Initial Samples][image1]

After that I applied the given Perspective Transform and show here that the rock sample also shows a nice blip in the transform.

![Applied Perspective Transform][image2]

After the transform, I applied a mask using thresholding on the image pixel's RGB, so I went with an upper and lower limit. The transformed pictures that I passed into the function were assigned arguments based on rock/terrain vision information. Here is a sample. In this sample we can see four different masks applied:



![Threshold Masking][image3]


Also, the color of the rock reaches a shadowed dark gold color due to the sunlight and shadows of the terrain that the camera captures, so I created the RGB threshold values to circumvent that issue so the robot would not get confused between a rock and the beige colored terrain.

These colors were decided upon by using an online color wheel chart and looking closely at the rock image in various lights.

thresh_rock = color_thresh(warped_rock,rgb_lower_limit=(140,110,0), rgb_upper_limit=(255,255,50))

In order to differentiate the navigable terrain from the rocks and the non-navigable terrain, I kept the light grey colored RGB code of 160,160,160.

threshed = color_thresh(warped,rgb_lower_limit=(160,160,160), rgb_upper_limit=(255,255,255))

Finally, for the non-navigable terrain, I wanted to keep the thresholding to match the color of the rocks so created a limit as close to black as possible. The Robot was getting confused if I set the upper limit too close to the terrain value and caused the fidelity to be reduced significantly.

The image above is set at an upper limit of RGB = 20,20,20.

threshed_walls = color_thresh(warped, rgb_lower_limit=(0,0,0), rgb_upper_limit=(20,20,20))

Now I want to show you what the images look like if I increaed the upper limit of the walls close to the terrain lower limit; where I make rgb_lower_limit =(130,130,130) for the walls.


![Threshold Masking][image4]

Here you see a wonderful mask for the walls at the top right pictures; however, fidelity is lost in the terrain masking (top left picture) and therein I had troubles creating a clear path for the robot. It was better left to be completely discrete and less ambiguous for the robot by keeping the rgb_upper_limit =(20,20,20). I was then able to create a clear path for the robot and the robot did not run into any walls. This helped me figure out how to address the image and the position to allow transformation, masking and assigned mapping for the terrain, rocks, and obstacles. However, I did have some ambiguity left that I would like to work on further as I refine this project.

After that, I changed coordinate systems from the rover centric coordinate system to world coordinates and applied the given arrow to the navigable terrain available in front of the robot. 

![Changed Coordinates with Arrow][image5]



#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
I populated the process_image() function by using code I had developed during the course and modified to manage identification of terrain, obstacles and rock samples into a worldmap. In order to provide real-time mapping for the robot mapping capability, I had to solve an ambiguity to populate the map with three colors to differentiate rock, navigable terrain and non-navigable terrain.

Since I didn't want the robot to possibly crash into the walls after mapping the wrong type of terrain, I solved an ambiguity by suggesting all ambiguous thresholded pixel would be considered non-navigable terrain.  I did that with a loop since I'm new to python and tried googling various ways to do it but found this was the best approach for me.

   for i in range(len(data.worldmap[:,:,0])):
        for j in range(len(data.worldmap[:,:,0])):
            if data.worldmap[i,j,0] > 0:
                data.worldmap[i,j,2] = 0


For the rocks I want to make sure it stays a green blip as I wasn't getting a green blip. It was just showing yellow. So I did the same thing for the green layer when I see a rock in my masking step.

   for i in range(len(data.worldmap[:,:,2])):
        for j in range(len(data.worldmap[:,:,2])):
            if data.worldmap[i,j,1] > 0:
                data.worldmap[i,j,2] = 0
                data.worldmap[i,j,0] = 0
   

I left the rest of the code as is and I saved a video under the name: **movie_dp_trial_100.mp4** in the **output folder**.


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

(Note: I filled out the perception_step() file in the perception.py script by using the code I had previously written and refering to the drive_rover.py for the Rover Class descriptions. However, after playing with the code (as you will read below), I changed the threshold of the walls to rgb_upper_limit(30,30,30). This helped me figure out how to address the image and the position to allow tranformation, masking and assigned mapping for the terrain, rocks and obstacles).

I was able to continously stay above 60% fidelity and map over 75% of the terrain with an average of 3 rocks identified.
I did get stuck several times in front of an obstacle and/or circling and tried to modify the code to help the robot become "unstuck" and continue on its mission. I mostly modified the perception.py to overcome this issue. Thought I did add some limiting code in the decision tree which I talk about in the next section.
I also added thresholds that would not validate transformed images for mapping using the roll and pitch angles, this helped the fidelity of map tracking stay high.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.
For the decision_step() I first played with the RoverSim in autonomous mode to see what the issues were. For the decision tree, I first played with the RoverSim in autonomous mode to see what the issues were. I drew out a tree on paper to further understand how the current tree given by the course team worked. I realized we could fix some of the code to help the robot navigate the terrain further. For example, if the robot is circling and mapping the same pixels several hundred times, how can we get the robot out from there? There are many possible solutions to this: tracking walls as to not circle and repeat mapping, or perhaps limiting and using starting position as a "reset" etc. This is something I would like to continue working on.

First off, I lose fidelity when the rover is turning. I know this is due to my coding where I apply a filter when I have the ambiguity of wall or navigable terrain. That filter is causing a disruption of continuous mapping allowed and I would like to fix that.

Secondly, I am not approaching each rock when I recognize it, I would like to add decisions to come close to the rock so to be able to pick it up. I can do this by using the masked filter of the rock to navigate toward the rock. I tried to add some coding to the decision tree so that when the robot saw a thresholded binary image of a rock; it would go into rock mode and stop and pick up the rock. Currently, the robot slows down to pick up the sample, but the sample moves out of perception and so the robot no longer sees the rock and then continues on its way to the next rock. As mentioned in the summary: I'm currently developing a Markov chain to improve the robots ability to see a sample rock and approach at a nominal velocity in order to slow down enough to stop and pick up the sample.

Finally, I have issues when I am stuck on slightly elevated terrain, I would like to propose to the robot to reverse and turn away from obstacles and try to go forward again. I did try to create a situation for the robot to turn around if the number of maxed mapped pixels in a given array was larger than 1000. This seemed to get the robot out of a bad situation such as circling, stuck on or at an obstacle and my next goal was to extend that code to help the robot not repeat its path.

I did this by first printing out the max value of the number of times one pixel was mapped in a particular rover image. = #print(np.amax(Rover.worldmap[:, :, 2])) Then I was able to set a threshold and see the type of masking the robot was seeing along with the max value. This caused me to rethink my obstacle threshold and keep it to a low number closer to a black color. It helped the robot stay out of a loop and also to navigate more terrain.

Occasionally the robot has gotten stuck or ran in a funny loop; this happens one out of several launches and I hope to completely eliminate that as I learn more coding techniques and spend more time thinking of the problem/solution.

I ran the simulator in autonomous mode on two different graphical choices. First I ran the simulation in a fast extremely low resolution of 640x480.
See picture below:
![Initial Simulator Settings][image6]
First I ran the testing on the lower graphics but I for more of a challenge- I ran to test fidelity on the higher graphics which allowed me to add various bits of coding to improve the robot's capability.
![Initial Simulator Settings][image7]

Overall I really enjoyed this project, it was my first programming experience with python and took me many hours but I enjoyed taking the extra time to learn, can't wait to improve upon this! My goal is to have the robot pick up all rocks and return back to center.







**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**






