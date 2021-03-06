layout: page
title: "How-to-Visualise-Data-in-Python-3-with-Plotly"
date: 2020-06-18 
categories: Data-Science

In this Python 3 tutorial I will describe how you can create data visualizations in python 3 with Plotly an open source graphing library.

# What is Plotly?

Plotly is an open source graphing library similar to matplotlib that allows you to create both online and offline graphs in Python 3. Below is example of a simple offline Bar Graph with Plotly

        import plotly.offline import iplot
        import plotly.graph_objs as go`

    # initiate the Plotly Notebook mode to use plotly.offline and inject the plotly.js source files into the notebook.
    init_notebook_mode(connected=True)

        data = [go.Bar(
                    x=['oranges', 'apples', 'mangos'],
                    y=[10, 20, 30]
            )]

        iplot(data, filename='bar')

![Plotly Bar Graph ](images/python_3_plotly_chloropleth-map2-600x400.png)

In this Python 3 Data visualization tutorial you will learn how to visualize the World Population data from World Bank using plotly bar graphs, plotly scatter plots and plotly choropleth maps. So let's get started.

## Step 1 : Installation and Version check

        pip install plotly pandas 


        from plotly import __version__
        from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
        import plotly.graph_objs as go

        import pandas as pd

        print(__version__)


        #It should print the ploty version as follows
        3.4.2

## Step 2 : Read the Data and Create a Python 3 Plotly Bar Graph of the World Population Data

You can download the data from the World Bank website or from my repo at https://github.com/Leo-G/Data-Visualisation-Tutorial-with-Python-3-and-Plotly

        #Read the excel file that contains the World Population and create a data frame using Python 3 Pandas
        df = pd.read_excel('dataSets/WorldBankPop2017.xlsx')

        #Check a sample of the data and columns in the Data frame
        df.head(5)

# Python 3 Pandas Plotly example

As you can see from the image above the data set has 5 columns excluding the row numbers. For  a Bar Graph we just need the Country and Population column.

        #Create a simple Plotly Bar Graph with the 'Country' and 'Population' columns
        data = [go.Bar(
         x= df['Country Name'],
         y= df['2017 [YR2017]'],
         width = 0.5

         )]

        # Add Title to the x and y axis respectively of your plotly graph
        layout = go.Layout(
         title='Countrywise Population Distribution',
         xaxis=dict(
         title='Tactics',
         ),


         yaxis=dict(
         title='Total Population',


         ),

        )

        #Create and plot the figure with Plotly
        fig = go.Figure(data=data, layout=layout)
        iplot(fig, filename='basic-bar')

# python 3 plotly bar graph tutorial

## Step 3 : Create a Python 3 Plotly Scatter Plot of the World Population

The above graph does not clearly indicate which countries have the highest population so lets try to visualize the data with a plotly scatter plot

        #Similar to the Bar Graph, create a scatter plot
        data2 = [go.Scatter(
         x= df['Country Name'],
         y= df['2017 [YR2017]'],
         mode = 'markers'
        )
         ]

        # Add Title to the x and y axis respectively of your plotly graph
        layout2 = go.Layout(
         title='Countrywise Population Distribution',
         xaxis=dict(
         title='Country',
         ),


         yaxis=dict(
         title='Total Population',


         ),

        )

        fig2 = go.Figure(data=data2, layout=layout2)
        iplot(fig2, filename='basic-scatter')

# python 3 plotly scatter graph

## Step 4 : Create a Python 3 Plotly Choropleth Map of the World Population

Although the outliers are much more clear in a scatter plot, the x-axis is clustered such that the two countries with the highest population India and China arn't seen . A better way to visualise this data would be a Choropleth Map  in which countries are shaded in proportion to their Population

        #Note when creating a Choropleth, you need to use the Country Code column based on which the countries in the map are shaded.

        data = [ dict(
         type = 'choropleth',
         locations = df['Country Code'],
         z = df['2017 [YR2017]'],
         text = df['Country Name'],
         showscale = True,


         marker = dict(
         line = dict (
         color = 'rgb(180,180,180)',
         width = 0.5
         ) ),
         colorbar = dict(
         autotick = False,

         title = 'Population'),
         ) ]

        layout = dict(
         title = '2017 World Population',
         geo = dict(
         showframe = False,
         showcoastlines = False,
         projection = dict(
         type = 'Mercator'
         )
         )
        )

        fig = dict( data=data, layout=layout )
        iplot( fig, validate=False, filename='WorldPOP')

# python 3 plotly choropleth map tutorial

There you go now you can clearly see that the two most populist countries are India and China.

You can view and download the complete Python 3 code for this tutorial along with the Jupyter notebook here
