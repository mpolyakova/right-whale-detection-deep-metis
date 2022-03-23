# Right Whale Detection Using Underwater Sensors 

## Abstract 
The North Atlantic Right Whale is a gentle giant who, quite matching his name, lives in the shallows along the Atlantic shoreline. These areas are an important shipping hub, causing a direct conflict with the whales. The population of North Atlantic Right Whales is below 400 in the world, and still decreasing, despite speed restrictions and fishing regulation. In the last 5 years, we've lost 34 whales. A decrease in calves born, as well as whale deaths from vessel strikes and entanglement means the population is unlikely to recover without more help.

NOAA requires a new solution to protect the whales from both vessel strikes and entanglements. Ships avoid each other by radar and AIS, and the whales need that too to prevent vessel strikes. For entanglement, a whale can frequently be saved if it is reached in time. Current methods of observation include aerial surveillance, and tourist sighting, but there is no method which covers a large area in a consistent way.  This method can be provided using existing buoys, and models to alert on whale presence by sound. 

Buoys are located all along the Atlantic coast to guide ships and indicate areas of interest. Currently, buoys already have near live data transmission capabilities, and are used for weather and sea level observations. Sound sensors can be places on buoys can provide real time data to aid training, and using a pre-trained deep learning model, identify nearby whales by sound. In a very future state, these sensors, along with the model, could be placed on ships, providing them their own whale radar.


## Design 
NOAA has collected sound data since 2009. Using underwater sound data, prelabeled with the presence of right whale upcall, I attempt to train a model to detect right whales using underwater sound. At this point, we are not attempting to interpolate the location of the whale, or the size of the whale pod, only a binary classification of presence or absence of whales.    

The ships location and speed is always known from their beacon. By being able to say whether a whale is nearby, we can reach out to ships directly indicating a whale in their path. Since sea regulations state that no ship can pass within 500 feet of a Right Whale, the ship must search and avoid the whale, therefore preventing a vessel strike.   
Additionally, buoy density is highest in areas of high traffic, so we would receive higher coverage where it is needed. Using the deep learning model, we would be able to alert the coast guard about a potential entrapment, potentially saving the whale.   

The cost of false negatives and false positives differs greatly for this problem. The cost of a false negative is potentially harming  and/or killing a rare whale. Since whale collisions can harm smaller vessels, there is potentially a monetary cost associated with false negatives as well. The cost of the false positive is monetary, either in time lost slowing down by shipping, or resources wasted to check potential entrapments. 


## Data 
The data is 2 second sound clips, prelabeled with the presence or absence of a right whale call. The clip is sampled at 2kHz, and therefore pick up only low frequencies (approximately up to 1024), however most whale calls fall within that range. The dataset contains 30K clips, with 5k with right whale upcalls, and 25K without. The clips without can contain boat noises, other whale sounds, and underwater species, as well as noise.  
The dataset is divided into 24K clips training set, 3K for validation and 3K for testing. The data is unbalanced with 1:5 ratio of right whale clips to other clips, and the data is noisy.   
The data used for training is precut, so we would need additional techniques in production to sample the incoming datastream for the start of signal. Potentially this can be bruteforced, cutting on overlapping windows very quickly and running the model on each, but can also be an additional model for signal detection. This is out of scope for this POC, however. 


## Algorithms 
### Data Processing 
I transformed the audio clips into spectrograms using Librosa. I transformed the raw frequencies into the logarithmic scales(melgrams) in both the intensity and the frequency domains, in order to better mimic mammal hearing patterns. I was not able to locate if whale hearing also follows this structure, however, I noticed that this pattern crispened a lot of the patterns in the spectrogram and made their intensity larger - but this also crispened noise.  

I created two versions of each spectrogram, one using dimensions suggested by raw data, which yielded a vertically heavy image - most of the information being stored on the frequency access(vertical), and few columns in the time domain. The original sample rate forced a maximum of 1024hz frequency, and 8 subdivisions of time. I transformed this data into a more square representation - 97 buckets of frequency, and 84 buckets of time. This made the visual easier for a human to interpret, and through experimentation, provided better results in the modeling stage.  
I saved the spectrogram as a colored 97 by 97 image, and read it back into keras.  

### Models 
After perusing literature(direct references in notebooks), I found successes in using pre-trained on imagenet networks to do sound classification using spectrograms. I attempted 3 types of models, based on 3 approaches I found: 
* Transfer learning using Resnet50 pretrained on imagenet, adding 2 dense layers at the end, an approach similar to one discussed in a very similar problem classifying humpback whale calls. 
* Transfer learning using VGG19 pretrained on imagenet, and adding 2 dense layers at the end, an approach from the Tropical sounds classification, and Grouper Sounds papers. 
* CNN building, looking at general sound classification blog posts.  
This literature is discussed in notebook 4.  

I trained each of these models on both spectrogram sets, and found that consistently the more square(called cb in the notebooks) performs slightly better. I defaulted to using it when iterating on models. 
I found that VGG19 performs worse that CNN on my input - it was longer to train, and after several attempts of adding more layers, I abandoned that approach first. 
CNN and Resnet both went through several iterations. I began with two Dense layers for both models, and attempted several different node sizes, from 100 to 1000, but did not find significant differences in either recall or accuracy with these differences. I found that the validation and training accuracies and recalls began to differ very quickly, so I added a smaller learning rate in the optimizer, and dropout layers for both models.  
I used early stopping and checkpoint callbacks to save the best version of my models. I experimented with validation accuracy vs validation loss for early stopping parameters, but didn't find a huge difference in performance.  

By default I started with 10 epoch, and some were cut at 5-6 epochs by early stopping. If the model looked promising at 10 epochs, I continued training. The final models used in testing were run freshly for 40 epochs, with the best model being saved by the checkpoint.  


I evaluated each type of model with the testing dataset. I evaluated two CNN and two Resnet models, and one VGG19 model. 

### Results 
The CNN and Resnet performed consistenly better than the VGG. In training, the CNN performed similarly to Resnet, and took less time to train, however, on a test dataset, all 3 resnet models performed slightly better than CNN on all metrics. This is likely indication that more experimentation is needed with CNN models, rather than the conclusion that Resnet transfer learning performs better than CNN cosistently on this problem. Overall, resnet recall on test data was between .72 and .89, while the CNN hovered around .79.  
There appears to be a tradeoff between precision and recall in this case - the resnet type model with higher precision is also the model with the lowerst recall.  

Since the cost of the false negative is much higher than that of false positives, we are interested in lowering false negatives, even at the expense of false positives. Because of this, the Resnet model with .89 recall is chosen. 



### Tools 
Librosa for sound to spectrogram processing 
Keras for building importing data and building and running models. 
Seaborn for visualization

### Communication
Each of the notebooks contains extensive explanations of through process and references. I completed at 5 minute presentation summarizing my goals and models. Slides are included in this repository. 

