# Stock price movement prediction - Deep Learning

## Introduction

Build a deep learning image classification model to predict stock price movements. The model will be trained using the images of Candlestick charts created with a sliding window of 5 days.

The overall system flow is shown below. 
* When the user enters a Yahoo Stock symbol
    1. The system will retrieve the last 5 days historical price
    2. Create a Candlestick chart 
    3. Use the trained model to predict the price movement for the following day
* The inference process will run daily and show the next day price predictions for the top 100 ASX stock (Australian Stock Exchange)
* System will also show the model performance on the test set generated during the inference process

![system-overview](system-overview.png)

The project can be broadly divided into three parts :  

I. Training Data Creation [COMPLETED]   
II. Build Models [IN-PROGRESS]   
III. Productionising, Inferencing and User Interface [NOT STARTED]   


## I. Training Data Creation
* The OHLC (Open-High-Low-Close) data for the stocks listed on the Australian Stock Exchange was used to create the candlestick charts (Training set)
* These charts were stored as images along with a target label that indicates whether the stock price moved up or down the following day
* Source files
    + **01_createDatasets.ipynb** : Jupyter notebook for creating the datasets. Also, show sample train and test set images
    + **functions_dataCreation.py** : Python file containing the functions used in the data creation process
* Coding notes
    + Used `mplfinance` package to create candlestick charts from OHLC CSV data.         
    + Customisation done using `matplotlib` [rcParams](https://matplotlib.org/3.2.1/tutorials/introductory/customizing.html#customizing-with-matplotlibrc-files)
    + Width adjustments done with help from (https://github.com/matplotlib/mplfinance/blob/master/examples/widths.ipynb)
    + Awesome .ipynb examples here (https://github.com/matplotlib/mplfinance/blob/master/examples)
    + Created RAMDisk for faster read/write of chart image files
    + Used `uint8` for `numpy` arrays and also for `create_dataset` in `h5py` .h5 file creation
    + Used `multiprocessing` pool function to spawn `mp.cpu_count()` simultaneous process


## II. Build Models
The overall plan is to use Tensorflow-Keras to test three types of Deep Learning models
1. Fully Connected Dense Networks
2. Convolutional Neural Networks
3. Transfer Learning


### 1. Fully Connected Dense Network
* Learning Rate Finder : LR is one of the important hyperparameters that we need to determine. It is used by the optimiser to adjust the model weights such that the loss is minimised
* Source files
    + **02_DenseNetwork_LRFinder.ipynb** :  I have used the [LR Finder class](https://www.jeremyjordan.me/nn-learning-rate/) to find the optimum LR range for different types of networks
    + **02_DenseNetwork_Experiments.ipynb** : The [Weights and Bias website](https://wandb.ai/amitagni/candlestick-simple?workspace=user-amitagni) was used to log the experiments conducted to reduce overfitting. Nothing worked.


### 2. Convolutional Neural Network
* Source files  
    + **030_CNN_Experiments.ipynb** : Some random unsucessful experiments [documented in wandb](https://wandb.ai/amitagni/candlestick-CNN?workspace=user-amitagni) 
    + **Time-Loss Evaluation** : How different model parameters impact Training time and Training loss
        + **031_Time-Loss-Experiments_Setup.ipynb**
        + **032_Time-Loss-Experiments_Evaluation.ipynb**
    + **033_Train_10000Epochs.ipynb** : Trained for 1.5 days with no significant improvement but PC crashed

.  
.  
.  

## Next Steps :
* Use tf.data to reduce memory usage
* Tensorflow profiling to investigate why GPU utilisation is less than 10%
* Tranfer learning
* Hyperparameter tuning
* 
* Things to try 
    + Try L1 regularisation, combined with L2
    + Use Large regularisation but train longer
    + Try other optimisation algorithms. Also do not rely on TF default  initialisations
    + Increasing the dataset size

.  
.  
.  

## III. Inferencing and User Interface
(NOT STARTED)  
.  
.  
.  
.  
.  
.  


## Draft Notes (To be Compiled)

#

* Fully Connected Network Build
    + Increased image size from 64x64 to 191x192 (Lowered underfit)
    + Batch Normalisation (Lower overfit)
        + `,keras.layers.BatchNormalization()`
    + Dropout Regularisation (Lower overfit)
        + `,keras.layers.Dropout(0.5)`
        + rate = keep_probability, lower for higher overfit
    + Activation layers : `kernel_initializer=tf.keras.initializers.` (Lower overfit)
        + he_normal() for `tanh` activations
        + GlorotNormal() for `relu` activations also called Xavier normal initializer
        + Both have normal and uniform versions

* Enabled Single Point precision (FP16) for better performance
* Installed `[tensorboard]`(https://www.tensorflow.org/tensorboard/get_started)
* used hParams 


### Some Random Notes on Dropout worsens performance

* Furthermore, be careful where you use dropout. It is usually ineffective in the convolutional layers, and very harmful to use right before the softmax layer.
* Dropout is a regularization technique, and is most effective at preventing overfitting. However, there are several places when dropout can hurt performance.
    + Right before the last layer. This is generally a bad place to apply dropout, because the network has no ability to "correct" errors induced by dropout before the classification happens. If I read correctly, you might have put dropout right before the softmax in the iris MLP.
    + When the network is small relative to the dataset, regularization is usually unnecessary. If the model capacity is already low, lowering it further by adding regularization will hurt performance. I noticed most of your networks were relatively small and shallow.
    + When training time is limited. It's unclear if this is the case here, but if you don't train until convergence, dropout may give worse results. Usually dropout hurts performance at the start of training, but results in the final ''converged'' error being lower. Therefore, if you don't plan to train until convergence, you may not want to use dropout.
    + Finally, I want to mention that as far as I know, dropout is rarely used nowaways, having been supplanted by a technique known as batch normalization. Of course, that's not to say dropout isn't a valid and effective tool to try out.



* The .h5 file sizes are huge 
    + Solved by setting dtypye to uint
    + `file.create_dataset('set_x', data=set_x,dtype='uint8')`
    + 5.4GB reduced to 700MB
    + So far no reduction in model performance (accuracy 60%)

* Also reduced the numpy array that is holding x and y from `int64` to `uint8`
    + `set_xy = (np.empty(shape=(loop_range),dtype = 'uint8')
            ,np.empty(shape=(loop_range,IMG_SIZE,IMG_SIZE,3)))`
    + CRASHED

* Other issues faced as mentioned above :
    + Had to use all the 6 cores using multiprocessing pool
    + Same random number was getting generated when the process was called. I have to pass seed, but couldnt get it to work as passing 3 variables in pool.map / pool.starmap was becoming a challenge
    + Solved by using the Date variable in the file name    
    + Earlier I used 64x64 images but the model seemed to stuck at 50% accuracy. Maybe more epochs would have helped ? not sure


* Tensorflow started giving memory error :
    + `Unable to allocate array with shape (156816, 36, 53806) and data type uint8`
    + Fixed after increasing page size.. not sure how
    + Alternate solution : `echo 1 > /proc/sys/vm/overcommit_memory` [NOT TRIED]
    + Source : https://stackoverflow.com/questions/57507832/unable-to-allocate-array-with-shape-and-data-type


* More memory issues :
    + `print(set_x.dtype)`  gives `uint8` as I have stored the images in that format as it also takes less space but any operation like subtraction and division, converts it to float32 and thus bloating up the space and hence the memory error
    + `print(set_x.nbytes/1024/1024)` gives  3.5GB
    + Solution is 
        + 1. Convert the array to float32 or float16 (but many methods expect float32)
        + Use keras.layers.BatchNormalization()
    + Side note : Types are changing to float due to tf.image.resize_images.


* Mixed precision
    + https://www.tensorflow.org/guide/mixed_precision
    + As my GPU is RTX2060 with compute capability of 7.5 I used it
        + `from tensorflow.keras.mixed_precision import experimental as mixed_precision`
        + `policy = mixed_precision.Policy('mixed_float16')`
        + `mixed_precision.set_policy(policy)`
    + Did not notice any impact on the performance

* Initially, the accuracy was stuck at 50%. Below were some of the things that helped in increasing it
    + The objective was to overfit the model i.e get the training accuracy to 99% 
    + Increased the shape of the images from 64x64 to 192x192. Tried other sizes like 
        + 128 (was better than 64)
        + 256 (Was increasing the array size)
        + Settled with 192x192 for now
    + The set_x/255. step was filling up the memory. So used Batch normalisation
    + Increased the training set to over 30K images (out of the available 60K + images)
        + The DATE_WINDOW reduced from 20 to 15

* Reducing Overfitting
    + kernel_regularizer=keras.regularizers.l2(0.001))
    + kernel_initializer='GlorotNormal'
        + Default is GlorotUniform (which is Xavier uniform)
        + According to this course by Andrew Ng and the Xavier documentation, if you are using ReLU as activation function, better change the default weights initializer(which is Xavier uniform) to Xavier normal by
        + Interesting papers linked here : https://stats.stackexchange.com/questions/339054/what-values-should-initial-weights-for-a-relu-network-be
        + If you dug a little bit deeper, you’ve likely also found out that one should use Xavier / Glorot initialization if the activation function is a Tanh, and that He initialization is the recommended one if the activation function is a ReLU. Source: https://towardsdatascience.com/hyper-parameters-in-action-part-ii-weight-initializers-35aee1a28404
    + kernel_initializer=tf.keras.initializers.he_normal()
        +  I will use this


* After adding `keras.layers.BatchNormalization()` after every Relu layer, the accuracy was initially very low as compared to without BN after every layer. But it is gradually increasing
    + After removing BN layers, the val acc for the first epoch increased from 25% to like 49% ????????


* Check memory occupied by int and float dtypes
```import numpy as np
def calcArrayMemorySize(array):
    return "Memory size is : " + str(array.nbytes/1024/1024) + " Mb"
    
print(calcArrayMemorySize(np.random.randint(0,255,size=(100,64,64,3))))
print(calcArrayMemorySize(np.random.random(size=(100,64,64,3))))
print(calcArrayMemorySize(np.random.random_sample(size=(100,64,64,3))))

Something wrong 
#Memory size is : 9.375 Mb
#Memory size is : 9.375 Mb
#Memory size is : 9.375 Mb
```

* Normalising Error
    + `set_x = set_x.astype("float32") / 255`
    + MemoryError: Unable to allocate 24.2 GiB for an array with shape (58842, 192, 192, 3) and data type float32



13th Jul 2020
* Cross checked data as validation performance not increasing
* Data set increased from 20 to 80 stocks, with start year of 2000

17th Jul 2020
* 15 layer model is giving average performance
* Try class weights with reg
* Add average model accuracy (predicting everything in majority class)



18th Jul 20
* For using the F1 metrics from tfa (tensorflowaddons), y should be one hot encoded
    + `metrics=['accuracy',tfa.metrics.FBetaScore(num_classes=3, average="micro", threshold=None )]`
    + And use `loss=tf.keras.losses.CategoricalCrossentropy()` 
* Changed class weights using `counts.sum()/counts/2`

21st Jul 20

    + Parameter size has no impact on model run time ?
        + 6 million parameters : 25s per epoch
        + 13 million : 25s per epoch
        + 28 million : 24s per epoch
        + CPU Util was 100%

