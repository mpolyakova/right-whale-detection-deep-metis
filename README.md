# Right Whale Detection Using Underwater Sensors 

## Abstract 
The North Atlantic Right Whale is a gentle giant who, quite matching his name, lives in the shallows along the Atlantic shoreline. These areas are an important shipping hub, causing a direct conflict with the whales. The population of North Atlantic Right Whales is below 400 whales in the world, and still decreasing, despite speed restrictions and fishing regulation. In the last 5 years, we've lost 34 whales. A decrease in calves born, as well as whale deaths from vessel strikes and entanglement means the population is unlikely to recover without more help.

NOAA requires a new solution to protect the whales from vessel strikes. Ships avoid each other by radar and AIS, and the whales need that too. Sensors can be places on buoys, which are common along dense shipping channels, provide real time data, and using a pre-trained deep learning model, identify nearby whales by sound. In a very future state, these sensors, along with the model, could be placed on ships, providing them their own whale radar.


## Design 
NOAA has collected sound data since 2009. Using underwater sound data, prelabeled with the presence of right whale upcall, I attempt to train a model to detect right whales using underwater sound. At this point, we are not attempting to interpolate the location of the whale, or the size of the whale pod, only a binary classification of presence or absence of whales.    

The ships location and speed is always known from their beacon. By being able to say whether a whale is nearby, we can reach out to ships directly indicating a whale in their path. Since sea regulations state that no ship can pass within 500 feet of a Right Whale, the ship must search and avoid the whale, therefore preventing a vessel strike.  
The data used for training is precut, so we would need additional techniques to sample the incoming datastream for the start of signal. Potentially this can be bruteforced, cutting on overlapping windows very quickly and running the model on each, but can also be an additional model for signal detection. 

## Data 
The data is 2 second sound clips, prelabeled with the presence or absence of a right whale call. The clip is sampled at 2kHz, and therefore pick up only low frequencies (approximately up to 1024), however most whale calls fall within that range. The dataset contains 30K clips, with 5k with right whale upcalls, and 25K without. The clips without can contain boat noises, other whale sounds, and underwater species, as well as noise. 

## Algorithms 


## 