---
layout:     post
title:      Logistic Regression.
date:       2020-04-01 12:32:18
summary:    Implementation of Logistic Regression from scratch using numpy. 
categories: jekyll pixyll
---


Lets implement Logistic Regression from scratch using the numerical python library numpy. 

{% highlight ruby %}
import random
import numpy as np
import warnings
import helpers.plt, helpers.dataset
from matplotlib import pyplot as plt
%matplotlib inline
warnings.filterwarnings('ignore')

print('Libraries imported')
{% endhighlight %}

Task 2: Hyperparameters


{% highlight ruby %}
lr = 0.3
batch_size = 300
iterations = 40
{% endhighlight %}

Task 3: Dataset


{% highlight ruby %}
(X_train, Y_train), (X_test, Y_test) = helpers.dataset.get_data()
{% endhighlight %}


{% highlight ruby %}
print('Shape of X_train:', X_train.shape)
print('Shape of Y_train:', Y_train.shape)
print('Shape of X_test:', X_test.shape)
print('Shape of Y_test:', Y_test.shape)
{% endhighlight %}


{% highlight ruby %}
helpers.plt.show_example(X_train[0], Y_train[0])
{% endhighlight %}


{% highlight ruby %}
helpers.plt.show_example(X_train[-1], Y_train[-1])
{% endhighlight %}

Task 4: A Mini Batch of Examples


{% highlight ruby %}
X_batch, Y_batch = helpers.dataset.get_random_batch(X_train, Y_train, 1)
print('X_batch shape:', X_batch.shape)
helpers.plt.show_example(X_batch[0], Y_batch[0])
{% endhighlight %}


{% highlight ruby %}
X_batch, Y_batch = helpers.dataset.get_random_batch(X_test, Y_test, 1)
helpers.plt.show_example(X_batch[0], Y_batch[0])
{% endhighlight %}


{% highlight ruby %}
X_batch, Y_batch = helpers.dataset.get_random_batch(X_train, Y_train, 10)
helpers.plt.show_ten_examples(X_batch, Y_batch, Y_batch)
{% endhighlight %}

Task 5: Create Model


{% highlight ruby %}
class LogisticModel:
    def __init__(self, num_features):
        self.W = np.reshape(np.random.randn((num_features)), (num_features, 1))
        self.b = np.zeros((1, 1))
        self.num_features = num_features
        self.losses = []
        self.accuracies = []
        
    def summary(self):
        print('=================================')
        print('Number of features:', self.num_features)
        print('Shape of weights:', self.W.shape)
        print('Shape of biases:', self.b.shape)
        print('=================================')
        
model = LogisticModel(num_features=784)
model.summary()
{% endhighlight %}

Task 6: Forward Pass


{% highlight ruby %}
class LogisticModel(LogisticModel):
    def _forward_pass(self, X, Y=None):
        batch_size = X.shape[0]
        Z = np.dot(X, self.W) + self.b
        A = 1. / (1. + np.exp(-Z))
        loss = float(1e5)
        if Y is not None:
            loss = -1 * np.sum(np.dot(np.transpose(Y), np.log(A)) + \
                           np.matmul(np.transpose(1-Y), np.log(1-A)))
            loss /= batch_size
        return A, loss
{% endhighlight %}

Task 7: Backward Pass


{% highlight ruby %}
class LogisticModel(LogisticModel):
    def _backward_pass(self, A, X, Y):
        batch_size = X.shape[0]
        dZ = A - Y
        dW = np.dot(np.transpose(X), dZ)/batch_size
        db = np.sum(dZ)/batch_size
        return dW, db
{% endhighlight %}

Task 8: Update Parameters


{% highlight ruby %}
class LogisticModel(LogisticModel):
    def _update_params(self, dW, db, lr):
        self.W -= lr * dW
        self.b -= lr * db
{% endhighlight %}

Task 9: Check Model Performance


{% highlight ruby %}
class LogisticModel(LogisticModel):
    def predict(self, X, Y=None):
        A, loss = self._forward_pass(X, Y)
        Y_hat = A > 0.5
        return np.squeeze(Y_hat), loss
    
    def evaluate(self, X, Y):
        Y_hat, loss = self.predict(X, Y)
        accuracy = np.sum(Y_hat == np.squeeze(Y)) / X.shape[0]
        return accuracy, loss
{% endhighlight %}


{% highlight ruby %}
model = LogisticModel(num_features=784)

model.summary()
X, Y = helpers.dataset.get_random_batch(X_test, Y_test, batch_size)
acc, loss = model.evaluate(X, Y)
print('Untrained model accuracy: {}, loss:{}'.format(acc, loss))
{% endhighlight %}


{% highlight ruby %}
X, Y = helpers.dataset.get_random_batch(X_test, Y_test, batch_size)
preds, _ = model.predict(X)
helpers.plt.show_ten_examples(X, Y, preds)
{% endhighlight %}

Task 10: Training Loop


{% highlight ruby %}
class LogisticModel(LogisticModel):
    def train(self, batch_size, get_batch, lr, iterations, X_train, Y_train, X_test, Y_test):

        print('Training..')
        self.accuracies = []
        self.losses = []
        
        for i in range(0, iterations):
            X, Y = get_batch(X_train, Y_train, batch_size)
            A, _ = self._forward_pass(X, Y)  
            dW, db = self._backward_pass(A, X, Y)
            self._update_params(dW, db, lr)
            
            X, Y = get_batch(X_test, Y_test, batch_size)
            val_acc, val_loss = self.evaluate(X, Y)
            self.accuracies.append(val_acc)
            self.losses.append(val_loss)
            
            print('Iter: {}, Val Acc: {:.3f}, Val Loss: {:.3f}'.format(i, val_acc, val_loss))
            
        print('Training finished.')
{% endhighlight %}

Task 11: Training the Model


{% highlight ruby %}
model = LogisticModel(num_features=784)

X, Y = helpers.dataset.get_random_batch(X_test, Y_test, batch_size)
print('Before training performance:', model.evaluate(X, Y))
model.train(
    batch_size,
    helpers.dataset.get_random_batch,
    lr,
    iterations,
    X_train, Y_train,
    X_test, Y_test
)
print('After training performance:', model.evaluate(X, Y))
{% endhighlight %}


{% highlight ruby %}
helpers.plt.plot_metrics(model)
{% endhighlight %}

{% highlight ruby %}
X, Y = helpers.dataset.get_random_batch(X_test, Y_test, batch_size)
preds, _ = model.predict(X)
helpers.plt.show_ten_examples(X, Y, preds)
{% endhighlight %}
