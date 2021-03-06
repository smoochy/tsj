# Python, R, & JavaScript Demo

This a small demo project that shows how values loaded into Python can be sent to the R environment, where some data analysis is done, and then values are returned to the Python environment. 

This example uses the Box and Jenkins airline data loaded into Python, R in order to generate a forecast about future data, and JavaScript to plot a chart show both the historical data and future data.

## Hat Tip
This project was build using the [Django Web Framework](https://www.djangoproject.com/), and built using the [Django Project Builder](https://github.com/prototypemagic/django-projectbuilder). JavaScript chart visualizations were done with the [dygraphs JS library](http://dygraphs.com/). A basic understanding on my part of doing time series analysis in R was made possible by the book [Introductory Time Series with R](http://www.amazon.com/dp/0387886974/). 

## Installing
To get this running on your own local machine, you will need Python and R installed. This has been tested on Mac OS X (Lion), with Python 2.7.1 and R version 2.15.2. 

First, clone the source: `git clone git@github.com:smoochy/tsj.git`.

The list of Python packages you need are listed in the requirements.txt file in the root folder of this project. You can install these by using the [pip package manager](https://pypi.python.org/pypi/pip). 

Once you have pip installed, run the following command inside of your project: `pip install -r requirements.txt`. 

This should install all of the listed packages into your Python environment, including rpy2, the Python bindings for the R environment. (Pro tip: to avoid cluttering your global Python environment with these packages, create a [virtualenv](http://www.saltycrane.com/blog/2009/05/notes-using-pip-and-virtualenv-django/) for your project.)

## Running
Django comes equipped with a small server you can run locally while in development. 

Inside the root directory of this project, run `python manage.py runserver`.

This will start up the server. You can view the demo in your browser at `http://localhost:8000/`.

If you have JavaScript disabled in your browser, don't do that. That said, make sure you have Java Applets disabled. :)

## Brain-dead Forecasting in R
Let's do some very simple forecasting in the R environment. Bring up an R shell in your environment of choice and enter the following commands:

```
# create a vector with values 1-36
valueVector <- 1:36
# load the values into a (monthly) timeseries
time_series <- ts(valueVector, freq=12)
# generate the forecast data for next 24 months
hw <- HoltWinters(time_series)
predict(hw, 24)
```

You will notice that R's forecasting has picked up on our contrived linear pattern, and predicts the next 24 months as values 37-60.

## Forecasting with R from Python
Let's do the same simple exercise, but this time from within a Python shell. 

To bring up a Python shell, run this inside your project: `python manage.py shell`.

The most difficult part here is importing all of the objects you need from R into the Python environment. Enter the following commands into the shell:

```python 
# import R stuff
import rpy2.robjects as ro
from rpy2.robjects.vectors import IntVector

# set up some basic R functions here
r = ro.r
ts = r['ts']
HoltWinters = r['HoltWinters']
predict = r['predict']

# Python list containing 1-36
value_list = range(1, 37)
# R needs a vector
value_vector = IntVector(value_list)
time_series = ts(value_vector, freq=12)
hw = HoltWinters(time_series)
predict(hw, 24)
# if you want forecasted values as a Python list
list(predict(hw, 24)) 
```

## Graphing
Now the trick is to get these values into a format so that JavaScript can plot them into a chart onto a web page. Dygraphs expects a CSV, presented as a string. The CSV looks like this:

Date,Historical,Forecast
1949-01-01,100,
1949-02-01,120,
... etc ...
1961-01-01,,450
1961-02-01,,430
... etc ...

Note that the earlier rows are missing Forecast values, and the later rows are missing Historical ones. This tells Dygraphs to plot two separate series on the same graph, visually distinguishing historical from forecast data.

## Exploring the Code
The code that generates the forecasts and builds the CSV string for the chart can be found in the `tsj_app/pythonr.py` file.

The Python and Django code that loads the web page (at http://localhost:8000) is inside of `tsj_app/views.py`.

The HTML for the home page is inside of `templates/index.html`. Note the empty div with an id of "chart" that will hold the chart.

The JavaScript to build the chart itself is located in `media/js/tsj/tsj.js`. This JavaScript file just loads the CSV data, builds the chart, and renders it into the div#chart element in the template.

## Wrapping Up
This project is loosely based on the code for an old and unfinished project of mine: [PredictCloud](http://www.predictcloud.com/). 

If you have any questions, feel free to get in touch: jpmcgaw (at) gmail.

Thanks!