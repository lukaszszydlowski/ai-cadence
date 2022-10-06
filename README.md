# AI Cadence
## The Experiment and motivation
The purpose of this experiment was to prove that a neural network can learn up correlation between bike movements and a cadence rider is stoking down the bike pedals during the ride. I was motivated by the fact 2 of my passions are computer since and cycling. For me, this was a perfect way to combine those two worlds together.
 ## Setting up and collecting data
 To collect training data along with target labels I used Garmin Edge 530 head unit attached to bike's handlebar with a rig mount. The head unit is capable of recording its movements acceleration in each of 3 axes with sample rate of 25 Hz and resolution up to 0,001 G. The usual cadence bike riders travel with varies between 60 and 120 stokes per minute (1-2 Hz). To correctly label data samples I used a cadence sensor attached directly to the bike's crank. A sample rate in this case was only 1 Hz. 
 
![Garmin Edge 530](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/Garmin%20Edge%20530.jpg)
![Cadence sensor](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/Cadence%20sensor.jpg)
## Utility 
To develop and test the utility I used Visual Studio Code and Garmin Monkey C environment and emulator. During my rides I managed to collect 380 minutes of labeled data. That gives a total of 22 800 samples. A single sample was 1 sec long and has 25 accelerometer readings form all 3 axes and 1 corresponding cadence label. I assumed this was enough to capture a characteristic of the movements while pedaling at certain cadence. All data was synchronised with the outside world using one of free to use key-value buckets as the Garmin don't supply programmers with file storage API.

![Monkey C emulator](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/Monkey%20C%20emulator.png)
## Data analysis and preprocessing
Before modeling up and training a network the samples needed preprocessing. I started with calculating FFT for 60 seconds long time frames and plotting the full 1 minute histogram of selected frames. During a visual analysis I didn't spot any strong correlation between points on the plot and recorder cadence. This might indicate that noise in the data. I also noticed that very strong, uncorrelated noise at 0 cadence (rider resting legs on crank or stopped). Eventually I decided to get rid of all 0 cadence samples from my data set. I thought it would help the algorithm to catch the correlation faster and perform better either on training as well as on validation data. 

![Sepctrogram](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/example%20spectrogram.jpg)
## Building and training the network
I started up modelling using very naive few Dense layers network. Not surprisingly, the network didn't learn any useful pattern and all it did was predicting a mean value over the entire training set. That clearly indicated that the model heavily underfitted the training set and/or algorithm quite easily converged to local minimum. I went through more complex network architectures, even with 1M+ trainable parameters. The conclusion was that adding more layers and units didn't improve the performance that much but significantly affected the training time. After days of experimenting with various architectures and fine tuning the models I ended up with yet simple but powerful network. The model that gave me acceptable results was as simple as 1 layer of Conv1D units to learn some useful features from input (signal spectrogram) and reduce the input size for the next layer. Then a layer with RNN units to give the network a chance to pick up relations in time sequence. The last layer was a single Dense neuron with RELU activation to predict the target value. The above is just a simplified description of what was implemented. In the real implementation I added a number on Dropouts and Batch Normalizations. I believe it helped to keep network more stable and speed up training a bit. 

![Model summary](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/model%20summary.png)
## Model evaluation
Finally, I approached evolution to validate and prove/disprove my initial assumption was right. During this phase I split collected samples into taring and validation sets with split ratio 80/20. As a metric I found MAE to be more interpretable by human than MSE. Despite this I used MSE as a loss function during all training process. A reason behind that was I believed that MSE penalises grater differences on predictions more than MAE and helps algorithm to converge faster and avoid local minimums (calculating  mean value over whole set). Unfortunately, the algorithm underperformed my expectations giving the final MAE of 7.8549 on validation set. However, plotting up the average error in function of cadence it can be seen that it preform quite well in cadence range between 70 and 115 giving error between 4-17%. Having in mind that this is the most observable range of cadence cyclists use, I may say the performance is decent. Also looking at the histogram of validation set target labels we can see that the most of the distribution is within a cadence range from 85 to 105. This suggest that I didn't collect enough data outside that range. Therefore the network seen only small numer of samples with unusual cadences during the training.

![Average error](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/average%20error.jpg)
![Histogram - Labels](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/histogram%20-%20labels.jpg)
## Conclusion
The experiment proved that cadence can be calculated this way as a function of accelerometer readings. Of course this method cannot be used in real live applications due to its limitations and error. I think the further signal denoising and processing along with training with much larger data sets would improve the results. Also placing a head unit on a seat post or closer to sadle (eg. on a top tube) would help to gather cleaner, less noisy data with more pronounced usable signal. 
