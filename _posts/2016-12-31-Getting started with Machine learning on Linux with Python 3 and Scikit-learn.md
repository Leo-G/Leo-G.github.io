In this Tutorial I will describe how you can get started with Machine Learning on Linux using Scikit-Learn and Python 3

Introduction
What is Machine Learning ?
Machine Learning is a way in which a Computing System like your Linux Computer can predict an output by learning from a sample set of Input Data.

For example
Predicting  an Unknown animal or flower based on it's features

While building a Machine Learning program you will need to  provide the program with a sample data set from which it can learn, Say features of known types of flowers. You can then use the program on data from an unknown type of flower to  predict it's type.

There are many types of Machine Learning Algorithm's which can be used to solve different types of problems. But since this is a getting started I will describe how you can solve a Simple Classification problem with Machine Learning using the "Decision Tree Classifier" Algorithm from Scikit-Learn

Versions
I have tested this tutorial on Ubuntu 14.04 and Linux Mint 18 with Python 3.5

Installation
The best way to install Scikit-Learn is to use the Anaconda package installer from 'continuum.io'. The installer comes with a script which will install Scikit-Learn along with other packages and dependencies required for your Linux machine.

 
#Download the Script
$ wget https://repo.continuum.io/archive/Anaconda3-4.2.0-Linux-x86_64.sh 
#Run the script,  
$ bash Anaconda3-4.2.0-Linux-x86_64.sh
#Finally install Graphviz for visualing the flow of the Program
$ sudo apt-get install graphviz 
The above bash script creates an anaconda3 folder which has all dependencies installed including a package manager called "conda" which you can use to install additional packages.


$ cd anaconda3/
$ source bin/activate
Step 1: Importing the Dataset

You will need two sets of Data. One which will be used to train the program and the other to test it.
Based on data from Wikipedia, I have created two artificial Datasets of Dogs to train and test our program

Training DataSet

Feeding(cups)	Weight(kg)	Height(cm)	Life Span(years)	Label
2	25	57	10	Labrador Retriever(0)
3	27	60	12	Labrador Retriever(0)
1	4.5	20	10	Shih Tzu(1)
2	5	27	15	Shih Tzu(1)

Testing Dataset

Feeding(cups)	Weight(kg)	Height(cm)	Life Span(years)	Label
2	25	56	9	Labrador Retriever(0)
1	4	27	14	Shih Tzu(1)

Step 2:Training the Model using the Decision Tree Classifier
The Decision Tree Classifier from Scikit-Learn will need two sets of input data to learn and predict the output. One is the features of the Dog and the other is the target or the type of dog corresponding to the features.


#Use the “python” command to switch to the python command line and then type the following
>>> from sklearn import tree
#Import the Dataset 
>>> features = [ [2,25,57,10], [3,27,60,12], [1,4.5,20,10], [2,5,27,15] ] 
>>> labels = [ 0, 0, 1,1]
#Create a Classifier Object
>>> classifier = tree.DecisionTreeClassifier() 
#Train the Classifier using the dataset 
>>> classifier.fit(features, labels)
Note: The labels have a numerical value instead of strings. 0 is for a Labrador Retriever and 1 is for a Shih Tzu
Step 3: Predicting the Output
Since it's a very small Dataset, the program should be able to predict the output without having to do much work. It should print a '0' for a Labrador Retriever and a '1' for Shih Tzu.


#Test Data for a Labrador Retriever(0)
>>> classifier.predict([2,25,56,9])
array([0])
#Test Data for a Shih Tzu(1)
>>> classifier.predict([1,4,27,14])
array([1])
Understanding how the decision was made with Visualisation
If you look carefully at the training Data set you will notice that the weight and height do not overlap. You can classify the breed of the dog by looking at its weight or height. Similary a decision tree classifier uses if-then statements to classify data. If the dog's weight or height is below a point then its a Shih Tzu. To check how the classifier made the decision you can visualize the Decision Tree with the Graphviz library we installed earlier.


#create a dot file from the python commandline
>>> with open('dogs.dot', 'w') as dotfile:
        tree.export_graphviz(
        classifier,
        dotfile,
        feature_names=['Feeding(cups)', 'Weight(kg)', 'Height(cm)', 'Life Span(years)' ])
#exit the command line and run the below command to convert the dot file to png
$ dot -Tpng dogs.dot -o dogs.png


Linux Machine Learning Tutorial

And this is how The Decision Tree classifier determined whether the dog is a Labrador or a Shih Tzu.

I took a very small Data Set since I feel it would be easier to understand, but in the real world a lot more data is required. For example below is the Decision Tree visualization of the iris dataset available at https://en.wikipedia.org/wiki/Iris_flower_data_set :

Linux Machine Learning Tutorial

Classification is just one type of Machine Learning, There are more and if you are interested then refer to the links at the end of the article.

Video of Installation and Classification is below:



You may also read how to visualise data in Python with Plotly

Reference Links:
youtube.com/watch?v=Gj0iyo265bc&list=PLOU2XLYxmsIIuiBfYad6rFYQU_jL2ryal
https://www.tensorflow.org/versions/r0.11/tutorials/image_recognition/index.html
http://scikit-learn.org/stable/tutorial/basic/tutorial.html
http://www.r2d3.us/visual-intro-to-machine-learning-part-1/
