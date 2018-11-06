
## Hyper parameters
### Learning rate
I found that starting with a larger learning rate and then decreasing it as needed during the training worked out the best. An initial `lr=0.007` was used. The reduction of the lr was done against the `val_loss` by using the Keras `ReduceLROnPlateau` callback function. Implemented as `callbacks.append(ReduceLROnPlateau(monitor='val_loss', factor=0.5, patience=7, verbose=1))`. By the end of all the training runs the learning rate had been reduced to `1.12e-06`.

### Batch size and steps
Learning was done on my laptop with a Quadrio M1000M, thus the batch size was small to account for this at `32`. A total of `35` steps were used for each epoch. This roughly encompasses `1/4` of the training data during a single epoch. The `10` validation steps covered `1/3` of the data per epoch.

### Epochs
Looking at the history results of the last run saved in the notebook 100 epochs were used. The model had already been trained twice for a total of 100 epochs (50+50) before started this training of 100 epochs. Generally I trained the model using 50 epochs as this would give me a good idea of how well it would converge. After 50 epochs I knew if I wanted to continue or try something new. Keras keeps the model in its current state and one can stop and restart training at any time as long as the model itself hasn't changed.


### Model Checkpoint
The model checkpoint Keras callback was also used to save the models' state whenever a new lower val_loss was achieved. Generally I would load and compare the 5 lowest models that were saved. Since the val_loss only checks against a portion of the validation data a model with a higher val_loss could perform better against the evaluation data. This is actually what was found with this final model. The final iteration of learning gave a val_loss of 0.023 whereas the lowest val_loss of the run was 0.0216. The 0.0216 gave a final result < 40% whereas the 0.023 > 40%.

[model]: ./model.png
## Model
Below is the model's graphical representation without the input or the softmax activation layers.
![][model]

I found that using a separable convolution with a stride of 2 gave me better results than using a pooling layer (max or average). I reused the same structure a few times. That is downsizing with a stride of 2 then using a 1x1 convolution to increase the complexity of the model. Each new 1x1 convolution would use a filter double the size of the previous one.

At the end of the encoding two 1x1 convolutions were used, but the second one only had a depth of 3. I found that finishing with a layer depth equal to the number of classes used for classification gave the best results. This also continues on to the decoding portion of the model. All convolutions done in the decoding process also used a filter size of 3 to match the number of classes. The depth of concatenated or upsampled layers are obviously based on the preceding layers and did not follow this rule.

The 2 2x2 convolution layers were sufficient in extracting the necessary important features of the input to allow for enough complexity in future layers to combine and produce the output. As such, I also found that I did not end up having to use regular convolution as multiple layers of separable convolution was sufficient. Using fully connected layers in the middle was tried, but did not yield necessary improvements and given the added complexity increased compute resources.

Also by strictly using the separable convolution throughout the model the training memory and computation requirement was lessened.

During the creation of the model many different layers were tried including Max Pooling, Avg Pooling, Dropout with different orders.

[hist]: ./hist.png
[imghero]: ./imghero.png
[imgopen]: ./imgopen.png
[imgpeople]: ./imgpeople.png
[results]: ./results.png
## Results
The model can be found in `./data/weights`.

As touched on in the Hyper Parameters section the model training was done in multiple runs. Here is the last history graph from the last run. As can be seen, the model had already gotten to a low val_loss of ~0.03 before starting and does not look like it improved a lot, but did a lot of jumping around. I decided to let it continue given the relatively low steps per epoch and validation as this will induce a certain amount of erratic behavior.
![][hist]

Below are some of the sample images given for grading the model and finally the results. As can be seen the model does a very good job at recognizing the hero when they are the only person in the field of view. The hero IoU is at 88% with zero false positives or negatives.

The model would still need more training for finding the hero from far away as the IoU was only 21% and it had some false negatives. Also when no hero is present there are some false positives. I believe that since other people are quite general to get the model to perform better more data would be required. I would also venture that potentially simplifying the model for people classification might give better results as it would give it a more generalized view of them.

I think reducing the amount of false positives would be quite hard as the hero does share a lot of similarities with regular people. Perhaps creating and integrating a custom activation function for the hero would be beneficial. This could be based on color/size/etc. Not too sure how I would go about this though.

This model is fairly limited in terms of use as it will strictly recognize just the one hero, and only people. Not interchangeable with other heroes or things or animals.

![][imghero]
![][imgpeople]
![][imgopen]
![][results]
