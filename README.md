# Object Detection in a Game
[This game](https://gyazo.com/306bc6cd46cf6b059c5ca289b07664d6) is notorious for its pixelated graphics, huge game ticks (effectively a 0.6s lag coded into the game), and unmistakable grid tiles. For CV applications, these characteristics are good. The blocky graphics means it should be easier to classify ([superpixelation](http://ttic.uchicago.edu/~xren/research/superpixel/) [works](http://ttic.uchicago.edu/~xren/research/superpixel/)), the large grids means that detection with a neural network can be brute forced because of the small number of locations to check, and the long ticks are generous with computation time permitted. 

## Detection with Thresholding
Easy: Color of object of interest is significantly different from background. For example, [finding a banker.](https://gyazo.com/e58b13e7b5f8e94029eaf9d0a1f9a8ee)

Harder: Colors of object of interest and background are similar. For example, [finding goblins.](https://gyazo.com/33ac61fe3f647bdde9bddaa6c0398c45)
It still found the goblins, but this highlights the weakness of color detection. In this MMORPG, I have no control over the colors that other players introduce. No matter how precise I make the thresholding, another player can wear colors of their choice.

Resource: Code taken from https://notebooks.azure.com/Microsoft-Learning/libraries/dev290x

## Detection with Convolutional Neural Network 
Goal: see if I can successfuly use a CNN can detect a goblin without giving a human as a false positive. 

Result: Success!  
 
-[Demo 1](https://gyazo.com/62ded2ce85fcf49efdf4d46dbf180826)

-[Demo 2](https://gyazo.com/e6248701c963cdeb19ee59f74ab6f368)
  
Before I got a model that worked, I had models that failed. I realized that from a top-down view, goblins look like old, hunched over humans. The model had difficulty differentiating humans from goblins (whereas recognizing the background was no problem at all), so I updated the training data to include more humans and less background, which had the effect of increasing the priority of the model training towards not incorrectly marking humans as goblins.
  
-[A failed model - At least it did not pick up the trees and fences](https://gyazo.com/515d5688d214f2d3c001e8f5ae46bfdb)

 
Comparison to thresholding: With thresholding I found the goblins by looking for the n-largest objects that passed the threshold. This meant that if I set n=3 and there were 8 goblins, only 3 would be marked. If I set n=8 and there were 3 goblins, 3 goblins and 5 green-ish things in the the background would be marked. The CNN approach is not limited by this. 

### Steps:
##### Step 1: Homography
The game does not offer a perfect straight down bird's eye view, so to split the screen into grids of equal size to take images for training data and object detection, homography was used. 
[Before](https://gyazo.com/73be4f3a2bcf759497c6ace0cc6f6616) versus [after.](https://gyazo.com/417e2edead71a2526dd30d0d56e6843b)

Before, there is a slight offset between the game tiles and the overlaid grid due to the camera angle.
After, the overlaid grid matches the tiles marvelously. 

About homography: https://www.learnopencv.com/homography-examples-using-opencv-python-c/

Another example:
[Left is the original image that a player would see. Right is after transformation.](https://gyazo.com/3b5bd74e1d315635736e81d6835e2303) The most noticeable is in the top left and right corners.

##### Step 2: Gather Training Data
In 'gather_data.py,' the click2crop function is run. The top left corner of a grid is doubleclicked and the content of that grid is saved. When gathering training data and performing inference, the camera angle is held constant. 

##### Step 3: Training
Using grayscale and rgb images converged on the same results. 

Model architecture: https://gyazo.com/72d21048232fe0de3d6396e800c7b88d

Result: https://gyazo.com/d104d5cfbc5cf4cfd8b37046153b726c

Loss Plot: https://gyazo.com/fa6e109ee96b576bd97cd86632c33996

##### Step 4: Apply Model to Real Time Detection
Inside a while loop, a function takes a screenshot. The screenshot is processed and split into 50x50 images that are fed into a model. The model classifies the segments and the screenshot is annotated with rectangles about detected objects. Finally the screenshot is displayed.
