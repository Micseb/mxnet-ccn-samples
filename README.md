# Traffic sign classification using Mxnet

In this notebook, we are going to classify german traffic sign using convolution neural network using mxnet.  The neural network will take a colored traffic sign image as input and tries to identify the meaning of the traffic sign. 

Although there are many deep learning frameworks (TensorFlow, Keras, Torch, Caffee), mxnet is gaining popularity due to its scalability across multiple GPU.

This notebook expects you to have a basic understanding of convolution operation, neural network, activation units, gradient decent, numpy and OpenCV. 

By the end of the notebook, you will be able to 
1.  Prepare dataset for training neural network
2.  Generate and Augment data to balance dataset
3.  Implement custom neural network architecture for a multiclass classification problem

## prerequisites
Note if you using conda environment, after you activate a environment, remember to install pip-
conda install pip. This will save you from lot of problems.

1. [Anaconda](https://www.continuum.io/downloads)
2. OpenCV - pip install opencv-python. You can also build from source. Note - conda install opencv3.0 didnt work.
3. [scikit learn] (http://scikit-learn.org/stable/install.html)
4. [Mxnet](http://mxnet.io/get_started/install.html)
5. Jupyter notebook - conda install jupyter notebook

## The dataset
To learn any deep neural network, we need data. For this notebook, we use a dataset that is already stored as numpy array, as it convenient and easy.  We can also load data from any image file. We will show that later in the notebook. 

The actual data set is located at  [here](http://benchmark.ini.rub.de/?section=gtsrb&subsection=dataset). Please read this page to understand the dataset better.

The pickled version of the data which we will be using is [here]
https://d17h27t6h515a5.cloudfront.net/topher/2017/February/5898cd6f_traffic-signs-data/traffic-signs-data.zip

The dataset consists of 39209 training samples and 12630 testing samples representing 43 different traffic signs (stop sign, speed limit, warning sign....).

Each data in the dataset is 32*32 size with three channel(RGB) and belongs to a particular image class. The image class is a number between 0 to 43.  The actual name of the class is given 'signnames.csv'. The 'signnames.csv' of CSV contains the mapping between the sign name and the class numbers. 

The code for loading the data is below

```python
import pickle

# TODO: Fill this in based on where you saved the training and testing data
training_file = "traffic-data/train.p"
validation_file =  "traffic-data/valid.p"

with open(training_file, mode='rb') as f:
    train = pickle.load(f)

with open(validation_file, mode='rb') as f:
    valid = pickle.load(f)
    
X_train, y_train = train['features'], train['labels']
X_valid, y_valid = valid['features'], valid['labels']
```
We are loading the data from a stored numpy array. The data is split between train and test set.  The train set contains the features of 39209 images of size 3*32 with 3 (R,G,B) channels . Hence the numpy array dimension is of 39209 * 32 X 32 X 3 array. So X_train is of dimension 39209 * 32 X 32 X 3. The y_train is of dimesion 39209*1 and contains a number between 0-43 for each image.

Next, we load the file that maps each image class to actual names.
```python
# The actual name of the classes are given in separate file. Here we load the csv file which allows mapping from classes/labels to 
# file name
import csv
def read_csv_and_parse():
    traffic_labels_dict ={}
    with open('signnames.csv') as f:
        reader = csv.reader(f)
        count = -1;
        for row in reader:
            count = count + 1
            if(count == 0):
                continue
            label_index = int(row[0])
            traffic_labels_dict[label_index] = row[1]
    return traffic_labels_dict
traffic_labels_dict = read_csv_and_parse()
print(traffic_labels_dict)
```
we can see there are 43 labels for the 43 image classes.  For example,
0 image class is actually speed limit(20 km/h).

```python
{0: 'Speed limit (20km/h)', 1: 'Speed limit (30km/h)', 2: 'Speed limit (50km/h)', 3: 'Speed limit (60km/h)', 4: 'Speed limit (70km/h)', 5: 'Speed limit (80km/h)', 6: 'End of speed limit (80km/h)', 7: 'Speed limit (100km/h)', 8: 'Speed limit (120km/h)', 9: 'No passing', 10: 'No passing for vehicles over 3.5 metric tons', 11: 'Right-of-way at the next intersection', 12: 'Priority road', 13: 'Yield', 14: 'Stop', 15: 'No vehicles', 16: 'Vehicles over 3.5 metric tons prohibited', 17: 'No entry', 18: 'General caution', 19: 'Dangerous curve to the left', 20: 'Dangerous curve to the right', 21: 'Double curve', 22: 'Bumpy road', 23: 'Slippery road', 24: 'Road narrows on the right', 25: 'Road work', 26: 'Traffic signals', 27: 'Pedestrians', 28: 'Children crossing', 29: 'Bicycles crossing', 30: 'Beware of ice/snow', 31: 'Wild animals crossing', 32: 'End of all speed and passing limits', 33: 'Turn right ahead', 34: 'Turn left ahead', 35: 'Ahead only', 36: 'Go straight or right', 37: 'Go straight or left', 38: 'Keep right', 39: 'Keep left', 40: 'Roundabout mandatory', 41: 'End of no passing', 42: 'End of no passing by vehicles over 3.5 metric tons'}
```
 ## Visualization
The below code will help to visualize the image along with the labels(images classes)

```python
# Data exploration visualization
# This gives better understanding of the data

# Data exploration visualization
# This gives better understanding of the data

import matplotlib.pyplot as plt
from matplotlib.figure import Figure
# Visualizations will be shown in the notebook.
%matplotlib inline

#This functions selects one image per class to plot
def get_images_to_plot(images, labels):
    selected_image = []
    idx = []
    for i in range(n_classes):
        selected = np.where(labels == i)[0][0]
        selected_image.append(images[selected])
        idx.append(selected)
    return selected_image,idx
 
# function to plot the images in a grid    
def plot_images(selected_image,y_val,row=5,col=10,idx = None):     
    count =0;
    f, axarr = plt.subplots(row, col,figsize=(50, 50))
   
    for i in range(row): 
         for j in range(col):
                if(count < len(selected_image)):
                    axarr[i,j].imshow(selected_image[count])
                    if(idx != None):
                        axarr[i,j].set_title(traffic_labels_dict[y_val[idx[count]]], fontsize=20)
                axarr[i,j].axis('off')
                count = count + 1
           
selected_image,idx = get_images_to_plot(X_train,y_train)
plot_images(selected_image,row=10,col=4,idx=idx,y_val=y_train)
```
The visualized traffic sign with their labels
![Alt text](images/vis.png?raw=true "traffic sign visualization")

## The data augmentation
```
If you explore the dataset with below code, there is a problem

```python
#count the number of data associated with each label. We can the data is not evenly distributed across the label
print(np.bincount(y_train))
```

```python
[ 210 2220 2250 1410 1980 1860  420 1440 1410 1470 2010 1320 2100 2160  780
  630  420 1110 1200  210  360  330  390  510  270 1500  600  240  540  270
  450  780  240  689  420 1200  390  210 2070  300  360  240  240]
```

There is not an equal representation of the images in the training set. The number of images of the class label 0 ('speed limit 20 km/h)' is 210 and the number of images of the class label 1 ('speed limit 30 km/h)' is 2220. This classes will cause neural network to favor class label 1 compared to class label 0.

In order to balance the dataset, we need to generate additional augmented data. We can augment data with various parameters, but for sake of simplicity, we are going to translate(move the image by random value in x, y dimensions) the image. Below is the code for data augmentation. 

```python
def random_trans(image, trans_range):
    rows,cols,_ = image.shape;
    tr_x = trans_range * np.random.uniform() - trans_range / 2
    tr_y = trans_range * np.random.uniform() - trans_range / 2
    Trans_M = np.float32([[1, 0, tr_x], [0, 1, tr_y]])
    image_tr = cv2.warpAffine(image, Trans_M, (cols, rows))
    return image_tr
```
The line Trans_M = np.float32([[1, 0, tr_x], [0, 1, tr_y]]) forms the translation matrix which is used to transalte the image"

Below is the code for balancing the dataset

```python
# Code for balancing dataset
def get_additional(count, label, X_train, y_train):
    selected = np.where(y_train == label)[0]
    counter = 0;
    m = 0;
    # just select the first element in selected labels
    X_mqp = X_train[selected[0]]
    X_mqp = X_mqp[np.newaxis, ...]
    while m < (len(selected)):
        aa =  random_trans(X_train[selected[m]],20)
        # ignore the first element, since it already selected
        X_mqp = np.vstack([X_mqp, aa[np.newaxis, ...]])
        if (counter >= count):
            break
        if (m == (len(selected) - 1)):
            m = 0
        counter = counter + 1
        m = m + 1
    Y_mqp = np.full((len(X_mqp)), label, dtype='uint8')

    return X_mqp, Y_mqp

def balance_dataset(X_train_extra, Y_train_extra):
    hist = np.bincount(y_train)
    max_count = np.max(hist)
    for i in range(len(hist)):
        X_mqp, Y_mqp = get_additional(max_count - hist[i], i, X_train, y_train)
        X_train_extra = np.vstack([X_train_extra, X_mqp])
        Y_train_extra = np.append(Y_train_extra, Y_mqp)
        
    return X_train_extra,Y_train_extra

X_train_extra,Y_train_extra =X_balance_dataset(X_train,y_train)
print(Y_train_extra.shape)
print(X_train_extra.shape)
```
## Prepared dataset

X_train_extra and Y_train_extra  make the actual training dataset. For sake of simplicity, we will use the testing set as validation set and we will use some real images for the purpose of testing. You can also generate a validation set by splitting the training data into train and validation set.
Below is the python code for doing it.

```
#split the train-set as validation and test set
from sklearn.model_selection import train_test_split
X_train_set,X_validation_set,Y_train_set,Y_validation_set = train_test_split( X_train_extra, Y_train_extra, test_size=0.02, random_state=42)
```
## The actual training

Now, enough of preparing out dataset. Lets actually code the neural network up. The neural code is actually small and simple , thanks to mxnet symbol api. 

```python
data = mx.symbol.Variable('data')
conv1 = mx.sym.Convolution(data=data, pad=(1,1), kernel=(3,3), num_filter=24, name="conv1")
relu1 = mx.sym.Activation(data=conv1, act_type="relu", name= "relu1")
#pool1 = mx.sym.Pooling(data=relu1, pool_type="max", kernel=(2,2), stride=(2,2),name="max_pool1")
# second conv layer
conv2 = mx.sym.Convolution(data=relu1, kernel=(3,3), num_filter=24, name="conv2", pad=(1,1))
relu2 = mx.sym.Activation(data=conv2, act_type="relu", name="relu2")
pool2 = mx.sym.Pooling(data=relu2, pool_type="max", kernel=(2,2), stride=(2,2),name="max_pool2")
#
#conv3 = mx.sym.Convolution(data=pool2, kernel=(5,5), num_filter=64, name="conv3")
#relu3 = mx.sym.Activation(data=conv3, act_type="relu", name="relu3")
#pool3 = mx.sym.Pooling(data=relu3, pool_type="max", kernel=(2,2), stride=(2,2),name="max_pool3")
# first fullc layer
flatten = mx.sym.Flatten(data=pool2)
fc1 = mx.symbol.FullyConnected(data=flatten, num_hidden=500, name="fc1")
relu3 = mx.sym.Activation(data=fc1, act_type="relu" , name="relu3")
# second fullc
fc2 = mx.sym.FullyConnected(data=relu3, num_hidden=43,name="final_fc")
# softmax loss
mynet = mx.sym.SoftmaxOutput(data=fc2, name='softmax')
```

Lets break the code a bit
```python 
data = mx.symbol.Variable('data')
```
It creates a data layer(input layer) that actually holds the dataset while training.

```python
conv1 = mx.sym.Convolution(data=data, pad=(1,1), kernel=(3,3), num_filter=24, name="conv1")
```
The conv1 layer performs a convolution operator on the image and is connected to the data layer.

```python
relu2 = mx.sym.Activation(data=conv2, act_type="relu", name="relu2")
```
The relu2 layer performs non linear activation on the input and is connected to convultion 1 layer.

```python
pool2 = mx.sym.Pooling(data=relu2, pool_type="max", kernel=(2,2), stride=(2,2),name="max_pool2")
```

The max pool layer performs a pooling operation(droping some pixels and reducing image size) on the previous layer's output(relu2).

A neural network is like lego block, so we repeat some of the layers (to increase the learning capacity of model) followed by a dense layer. A dense layer is a fully connected layer where every neuron from the previous layer is connected to every neuron in a dense layer.

```python
fc1 = mx.symbol.FullyConnected(data=flatten, num_hidden=500, name="fc1")
```

This layer is followed by again a fully-connected layer with 43 neurons, each neuron representing a class of the image. Since the output from the neuron is real valued, but our classification requires a single integer as output, we use another activation function. This will make the output of one particular out of 43 neurons as 1 and remaining 1 neuron as zero.

```python
fc2 = mx.sym.FullyConnected(data=relu3, num_hidden=43,name="final_fc")
# softmax loss
mynet = mx.sym.SoftmaxOutput(data=fc2, name='softmax')
```

## Tweaking training data.
A neural network takes a lot of time and memory to train. In order to train neural network efficiently, we split a dataset into batches that fit into memory easily, so we split into batches of 64. 

Also, we train the normalize the value of image color (0-255) to the range of 0 to 1. This helps the learning algorithm to converge faster. You can read about the reasons to noramlise the input online

Below is the 
```python

batch_size = 64
X_train_set_as_float = X_train_reshape.astype('float32')
X_train_set_norm = X_train_set_as_float[:] / 255.0;

X_validation_set_as_float = X_valid_reshape.astype('float32')
X_validation_set_norm = X_validation_set_as_float[:] / 255.0 ;


train_iter =mx.io.NDArrayIter(X_train_set_as_float, y_train_extra, batch_size, shuffle=True)
val_iter = mx.io.NDArrayIter(X_validation_set_as_float, y_valid, batch_size,shuffle=True)


print("train set : ", X_train_set_norm.shape)
print("validation set : ", X_validation_set_norm.shape)


print("y train set : ", y_train_extra.shape)
print("y validation set :", y_valid.shape)
```

## Lets train the network
We are training the network using GPU since its faster. We are training the network for 10 epoch "num_epoch = 10".  A single pass through the training set is  called as one epoch. We also prediocally store the trained model in a json file. We also measure the 
train and validation accuracy. For windows users, use only cpu. Mxnet has bug in gpu implemention for windows.

Below is the code 
```python
#create adam optimiser
adam = mx.optimizer.create('adam')

#checking point (saving the model). Make sure there is folder named models exist
model_prefix = 'models/chkpt'
checkpoint = mx.callback.do_checkpoint(model_prefix)
                                       
#loading the module API. Previously mxnet used feedforward (deprecated)                                       
model =  mx.mod.Module(
    context = mx.gpu(0),     # use GPU 0 for training if you dont have gpu use mx.cpu(). 
    symbol = mynet,			 
    data_names=['data']
   )
                                       
#actually fit the model for 10 epochs. Can take 5 minutes                                      
model.fit(
    train_iter,
    eval_data=val_iter, 
    batch_end_callback = mx.callback.Speedometer(batch_size, 64),
    num_epoch = 10, 
    eval_metric='acc', # evaluation metric is accuracy. 
    optimizer = adam,
    epoch_end_callback=checkpoint
)
```

## load the trained model from file system
Since we have check pointed the model during training, we can load any epoch and check its classification power. Below we are loading the 10 epoch. We also set binidng in the model loaded with training false, since we are using this network for testing and not training. We also reduce the batch size of input from 64 to 1 (data_shapes=[('data', (1,3,32,32))) , since we are going to test it on a single image. You can use the same technique to load any other pretrained machine learning model.

```python
#load the model from the checkpoint , we are loading the 10 epoch
sym, arg_params, aux_params = mx.model.load_checkpoint(model_prefix, 10)

# assign the loaded parameters to the module
mod = mx.mod.Module(symbol=sym, context=mx.cpu())
mod.bind(for_training=False, data_shapes=[('data', (1,3,32,32))])
mod.set_params(arg_params, aux_params)
```

## prediction
We are using the load model for prediction. We convert the some traffic sign image(turn-left-ahead2.jpg) and try to predict their label. Below is the image I downloaded from google

![Alt text](images/turn-left-ahead2.jpg?raw=true "test image")


```python
#Preidcition for random traffic sign from internet
from collections import namedtuple
Batch = namedtuple('Batch', ['data'])

#load the image , resizes it to 32*32 and converts it to 1*3*32*32 
def get_image(url, show=False):
    # download and show the image
    img =cv2.imread(url)
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    if img is None:
         return None
    if show:
         plt.imshow(img)
         plt.axis('off')
    # convert into format (batch, RGB, width, height)
    img = cv2.resize(img, (32, 32))
    img = np.swapaxes(img, 0, 2)
    img = np.swapaxes(img, 1, 2) #swaps axis to make it 3*32*32
    #plt.imshow(img.transpose(1,2,0))
    #plt.axis('off')
    img = img[np.newaxis, :] # Add a extra axis to the image so it becomes 1*3*32*32
    return img

def predict(url):
    img = get_image(url, show=True)
    # compute the predict probabilities
    mod.forward(Batch([mx.nd.array(img)]))
    prob = mod.get_outputs()[0].asnumpy()
    # print the top-5
    prob = np.squeeze(prob)
    prob = np.argsort(prob)[::-1]
    for i in prob[0:5]:
        print('class=%s' %(traffic_labels_dict[i]))

predict('traffic-data/turn-left-ahead2.jpg',)
```