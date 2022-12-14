# AI Cadence
## The Experiment and motivation
The goal of this experiment was to prove that a neural network can learn the correlation between bike displacements and a cadence rider is stoking down the pedals during the ride. I was motivated by the fact 2 of my passions are computer science and cycling. For me, this was a perfect way to combine those two worlds together.
## Setting up and collecting data
To collect training data and target labels I used a Garmin Edge 530 head unit attached to the handlebar with a rig mount. The head unit is capable of recording acceleration along each of 3 axes with sample rate of 25 Hz and sensitivity up to 0,001 G. The usual cadence cyclists travel with, varies between 60 and 120 strokes per minute. In order to correctly label data samples I used a cadence sensor attached directly to the crank. A sample rate in this case was only 1 Hz. 
 
![Garmin Edge 530](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/Garmin%20Edge%20530.jpg)
![Cadence sensor](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/Cadence%20sensor.jpg)
## Utility 
To develop and test the utility I used Visual Studio Code, Garmin Monkey C environment and emulator. During my rides I managed to collect 380 minutes of labeled data. That gives a total of 22 800 samples. A single sample was 1 sec long and has 25 accelerometer readings from all 3 axes and 1 corresponding cadence label. I assumed this was enough to capture a characteristic of the displacements while pedaling at a certain cadence. All data was synchronized with the outside world using a cloud based key-value bucket, as the Garmin don't supply programmers with file storage API.

![Monkey C emulator](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/Monkey%20C%20emulator.png)
## Data analysis and preprocessing
Before modeling up and training a network the samples needed preprocessing. I started with calculating FFT for 60 seconds long time frames and plotting the spectrogram of selected frames along axis X. During a visual analysis I didn't spot any strong correlation between points on the plot and recorder cadence. This might indicate that there was a noise in the data. I also noticed strong, uncorrelated noise at 0 cadence (rider resting legs on crank or stopped riding). Eventually I decided to get rid of all 0 cadence samples from my data set. I thought it would help the algorithm to catch the correlation faster and perform better either on training as well as on validation data. 

![Sepctrogram](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/example%20spectrogram.jpg)
## Building and training the network
I used PyCharm IDE, Python and TensorFlow to enhance model building. I started up modeling using a naive Dense layers network. Not surprisingly, the network didn't learn any useful pattern and all it did was predicting a mean value over the entire training set. That clearly indicated that the model underfitted the training set or algorithm converged to local minimum. I went through more complex architectures to the deep networks, even with 1M+ trainable parameters on the other extreme. In this case the network often memorized the training data due to overfitting. The conclusion was that adding more layers and units didn't improve the performance that much, but significantly affected the training time. Turning regression problem into clasiffication problem such as predicting cadence zones (low, normal, high) did not bring any interesting results as well. After days of experimenting with various architectures and fine tuning the models, I ended up with a simple but powerful network. The model that gave me acceptable results was as simple as 1 layer of Conv1D units to learn some useful features from input (signal spectrogram) and reduce the number of time steps in sequence for the next layer. Then a layer with RNN units to give the network a chance to pick up relations in time sequence (cadence reading from sensor has 1-2 seconds of delay). The last layer was a single Dense neuron with RELU activation to predict the target value. The above is just a simplified description of what was implemented. In the real implementation I added a number on Dropouts and Batch Normalizations. It helped to keep network training more stable and speed up training a bit. 

![Model summary](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/model%20summary.png)
## Model evaluation
Finally, I approached evaluation to validate and prove/disprove that my initial assumption was right. During this phase I used previously splitted collected samples into training and validation sets with split ratio 80/20. As a metric I found MAE to be more interpretable by humans than MSE was. Despite this I used MSE as a loss function during the training process. A reason behind that was that MSE penalizes greater errors on predictions more than MAE does and helps the algorithm to converge faster and avoid local minimums (calculating mean value over the whole set). The algorithm underperformed my expectations giving the final MAE of 7.8549 on validation set. 

![Evaluation](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/evaluation.png)

However, when plotting up the average error in function of cadence, it can be seen that it generalizes quite well in the cadence range between 70 and 115 giving error between 4-17%. Having said that this is the most observable range of cadence cyclists use, I may say the performance is decent. Also looking at the histogram of validation set target labels we can see that the most of the distribution is within a cadence range from 85 to 105. This suggests that I didn't collect enough data outside that range. Therefore the network has seen only a small number of samples with unusual cadences during the training.

![Average error](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/average%20error.jpg)
![Histogram - Labels](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/histogram%20-%20labels.jpg)
## Conclusion
The experiment proved that cadence can be calculated this way and the algorithm has some potential to improve. This method cannot be used in real life applications due to its limitations and error. I think the further signal denoising and processing along with training with much larger data sets would improve the results. The data augmentation seems to be not possible in that case. Secondly, placing a head unit on a seat post or somewhere closer to saddle (eg. on a top tube) would help to gather cleaner, less noisy data with more pronounced meaningful signals. Finally, adding readings from additional sensors such as speed and gyroscope could bring benefits and reduce prediction error even more.



![Lukasz](https://raw.githubusercontent.com/lukaszszydlowski/ai-cadence/main/pictures/lukasz.jpg)



