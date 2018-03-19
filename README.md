# RoboND Project (March 2018)
## Project: Search and Sample Return
### Divya Patel (OptimomEngineer)
---

**The goals / steps of this project are the following:**  

1) **Training / Calibration**  

* Download the simulator and take data in "Training Mode" ###Done
* Test out the functions in the Jupyter Notebook provided ###Done
* Add functions to detect obstacles and samples of interest (golden rocks) ###Done
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works. #Done
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission. #Done

2) **Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). #Done
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  #Done

[//]: # (Image References)

[image1]: /misc/initial_samples.png
[image2]: ./misc/Pers_transform.png
[image3]: ./misc/thresh1.png 
[image4]: ./misc/thresh2.png
[image5]: ./misc/arrow.png
[image6]: ./misc/simulator_settings.png
[image7]: ./misc/simulator_settings.png


## Writeup / README
### The following write up will provide insight as to how I wrote my code. 

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
I first started the Rover simulator and recorded a stream of data by moving the robot through the terrain.
I ran the functions provided in the notebook both on test images and also on a random image that I pulled from the recorded data.
Here is a sample of the image I pulled from recorded data along with the two test images I used.

![Initial Samples] [image1]

After that I applied the given Perspective Transform and show here that the rock sample also shows a nice blip in the transform.

![Applied Perspective Transform][image2]

Once we did that, I wanted to apply a mask, so I went with an upper and lower limit. The transformed pictures that I passed into the function were assigned arguments based on rock/terrain vision information. Here is a sample. In this sample we can see four different masks applied:


![Threshold Masking] [image3]


Since the color of the rock is gold not yellow, I added 50 to blue channel in upper limit. 

Also, the color of the rock reaches a shadowed gold color, reduced to a dark gold color, so I created the lower limit to represent that as not to get confused with a beige colored terrain. 

These colors were decided upon by using an online color wheel chart and looking closely at the rock image.

thresh_rock = color_thresh(warped_rock,rgb_lower_limit=(140,110,0), rgb_upper_limit=(255,255,50))

Second, I want to show the terrain that we can navigate through so I kept the 160,160,160 RGB code. 

threshed = color_thresh(warped,rgb_lower_limit=(160,160,160), rgb_upper_limit=(255,255,255))

Finally, for the walls, I wanted to keep it dark and so left it as close to black as possible. The Robot was getting confused if I set the upper limit too close to the terrain value and caused the fidelity to be reduced significantly. 

The image above is set at an upper limit of RGB = 20,20,20.

threshed_walls = color_thresh(warped, rgb_lower_limit=(0,0,0), rgb_upper_limit=(20,20,20))


Now I want to show you what the images look like if I increaed the upper limit of the walls close to the terrain lower limit; where I make rgb_lower_limit =(130,130,130) for the walls.

![Threshold Masking][image4]

Here you see a wonderful mask for the walls at the top right pictures; however, fidelity is lost in the terrain masking (top left picture) and therein I had troubles creating a clear path for the robot. It was better left to be completely discrete and less ambigious for the robot by keeping the rgb_upper_limit =(20,20,20). I was then able to create a clear path for the robot and the robot did not run into any walls. However, I did have some ambiguity left that I would like to work on further as I learn more in this class.


After that I changed coordinate systems and applied the given arrow to the navigatible terrain available in front of the robot.

![Changed Coordinates with Arrow][image5]



#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
I populated the process_image() function by using code I had developed during the course and modified to manage identification of terrain, obstacles and rock samples into a worldmap. 

For the terrain and walls, I'm was getting a purple/white overlap, but I wanted blue/red. Since I didn't want the robot to possibly crash into the walls, I solved that ambiguity by suggesting all purple is walls. I did that with a loop since I'm new to python and tried googling various ways to do it but found this was the best approach for me.

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
I filled out the perception_step() file in the perception.py script by using the code I had previously written and refering to the drive_rover.py for the Rover Class descriptions. This helped me figure out how to address the image and the position to allow tranformation, masking and assigned mapping for the terrain, rocks and obstacles.

I was able to continously stay above 60% fidelity and map over 75% of the terrain with an average of 3 rocks identified.
I did get stuck several times and tried to modify the code to help the robot become "unstuck" and continue on its mission.
#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.
For the decision_step() I first played with the RoverSim in autonomous mode to see what the issues were. 

First off, I lose fidelity when the rover is turning. I know this is due to my coding where I apply a filter when I have ambiguity of wall or navigable terrain. That filter is causing a disruption of continuous mapping allowed and I would like to fix that. However, for the assignment, the robot still passes mapping 40% at 60% fidelity.

Secondly, I am not approaching each rock when I recognize it, I would like to add decisions to come close to the rock so to be able to pick it up. I can do this by using the masked filter of the rock to navigate toward the rock.

Finally, I have issues when I am stuck on slightly elevated terrain, I would like to propose to the robot to reverse and turn away from obstacles and try to go forward again.

I ran the simulator in autonomous mode on two different graphical choices.
See picture below:

![Initial Simulator Settings][image6]

First I ran the testing on the lower graphics but I wanted more of a challenge so then I ran to test fidelity on the higher graphics.

![Initial Simulator Settings][image7]

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**




