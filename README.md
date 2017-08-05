
"With four parameters I can fit an elephant, and with five I can make him wiggle his trunk."
*John von Neumann*

![Example of deconvolved trace of a dataset without ground truth](https://github.com/PTRRupprecht/Spikefinder-Elephant/blob/master/figures/Stripe1.png)

### 0. Introduction

This is a program based on convolutional neural networks for spike detection from calcium traces, written by [@PTRRupprecht](https://github.com/PTRRupprecht) and [@unidesigner](https://github.com/unidesigner), as part of the Spikefinder coding challenge 2017 (http://spikefinder.codeneuro.org/). It is written in Python and Keras. The original version used for the competition submission contained also Matlab code (still in the repository) that was later replaced by Python code.


0. Install tensorflow and [Keras](https://keras.io/) in a Python 3 environment (e.g. [Anaconda]( https://www.continuum.io/downloads))

1. Download train and test data of the Spikefinder competition into folders relative to this root folder:
    spikefinder.test/
    spikefinder.train/

2. a) Start with the interactive Jupyter file `Demo.ipynb`

2. b) Start ipython or spyder and load the `demo.py`

Alternatively, we provide a [Docker file](/docker/) that can be used to run `Demo.ipynb` in a containerized version on any platform.

A concise description of the algorithms and the ideas behind it will be published soon in the context of a review paper associated with the Spikefinder coding challenge.

### 1. Instructions for Users (simple convolutional model)

For an interactive readme, open `Demo.ipynb`. Read on if you prefer to use `demo.py`:

We suggest to generate a local copy of this repository, open the file `demo.py` and execute the code step by step (Part I). In this code, an example dataset is used (an unknown dataset of 110 neurons with calcium recordings without ground truth).

This part uses a simple pre-trained convolutional neuronal network (CNN) to predict spikes from the calcium traces. 

### 2. Instructions for Users (using embedding spaces)

This section can be found in Part II of `demo.py` (or Part II of `Demo.ipynb`). To refine predictions, a pre-trained CNN is loaded as before, but retrained based on a selection of the training data. The selection is based on statistical properties of the calcium traces of the dataset to be analyzed. The idea of embedding spaces is explained below in further detail.

![Another example of deconvolved trace of a dataset without ground truth](https://github.com/PTRRupprecht/Spikefinder-Elephant/blob/master/figures/Stripe2.png)

### 3. Code Organization for Developers

To understand how the model was developed and how predictions for the Spikefinder competition were made, please refer to:

1) `run_basic_CNN.py` for the simple CNN model
2) `run_statEmbedding.py` for the model based on embedding spaces

Both are based on a simple neuronal network with three convolutional layers:

![Table with main layers of the basic CNN in Keras](https://github.com/PTRRupprecht/Spikefinder-Elephant/blob/master/figures/Figure4.png)

Windowsize of the input is 128 datapoints, corresponding to 1.28 sec. Filter sizes of the convolutional filters are 41, 21 and 7 pixel for conv1d_1, conv1d_2 and conv1d_3, respectively. No zero padding was used.
<br><br>

### 4. The idea behind the embedding spaces

The ten available datasets are different in terms of calcium indicator, signal-to-noise, neuron type and brain area and possibly temperature of the recording, resulting in different optimal convolutional filters for each dataset. We try to solve this problem by first answering the following questions: Which neurons/datasets have similar convolutional filters?

To answer this question, we fit a simple 2-layer convolutional network to a single neuron and then use this network to predict the spikes for all other neurons. In the figure (Subfig. A), row 55 e.g. shows how well spikes of neuron 55 can be predicted by the networks generated by all other neurons; column 55 shows how well the model generated via neuron 55 can predict spikes of other neurons.
Normalizing over columns, symmetrizing the matrix and averaging over datasets yields the distance matrix (Subfig. B, datasets indicated by numbers).
A PCA of the distance matrix yields an embedding space, of which the first two components are plotted (Subfig. C). Datasets being close to each other (e.g. 7 and 10) can predict each other's very well, whereas datasets distant from each other in space (e.g. datasets 5 and 4) are not good at predicting each other's mutual spikes.

![Predictive power of models fitted for single neurons](https://github.com/PTRRupprecht/Spikefinder-Elephant/blob/master/figures/EmbeddingSpaces_XX.jpg)

The next challenge is to map a neuron of a dataset of unknown properties onto the right location of this embedding space. To this end, we calculated statistical properties of the raw calcium time traces: Variance, kurtosis, skewness; autocorrelation value after 0.5, 1 and 2 seconds; generalized Hurst exponents 1-5; and the power spectral density at different frequencies between 0.1 and 3.6 Hz (Subfig. D). After averaging the values over datasets (Subfig. E), we used the two first principal components to generate a map of proximity in statistical property space (Subfig. F). This map was generated using the training dataets (numbers on the right side of the symbols), and test datasets were mapped into the PCA space (numbers below the symbols).

Next we trained a regressor to map the statistical poperties' embedding space to the predictive embedding space. For this we used a DecisionTreeRegressor from the scikit-learn package (mapping between Subfigs. C/F).

In total, this procedure spans an embedding space that allows to understand the mutual predictive power of different datasets, and the mutual distance between datasets in terms of their statistical properties.

It would probably require a larger collection of diverse datasets to make this embedding robust and good for any new datasets. In the 10 datasets given, there are some outliers (e.g. dataset 5), and if a new dataset is an outlier as well (which you cannot know beforehand easily), it will not be predicted well by any model that uses those 10 datasets in a selective or attentive manner.

Some results of the network applied to before unknown datasets are shown in [this blog post](https://ptrrupprecht.wordpress.com/2017/08/01/a-convolutional-network-to-deconvolve-calcium-traces-living-in-an-embedding-space-of-statistical-properties/).

