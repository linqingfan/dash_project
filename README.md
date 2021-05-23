# Template for Login Page on Dash/Plotly platform
![Alt text](https://github.com/linqingfan/dash_project/blob/main/covid19.png?raw=true "Example Covid 19")

## Setting up Dash User Management
First download the template source:
```
git clone https://github.com/Chris3691/Dash-User-Management
```
Already installed the required python packages with the following:
```
sudo pip3 install flask_login sqlalchemy flask_sqlalchemy PyMySQL dash_bootstrap_components pandas
```

Edit the following file config.txt:
```
[database]
con = mysql+pymysql://user:password@localhost/database_name
````
Create a database in mysql with the following command:
```mysql
CREATE DATABASE tgdata_db;
USE tgdata_db;
#Note password size has been changed to 88

CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) NOT NULL,
  `email` varchar(50) NOT NULL,
  `password` varchar(88) NOT NULL,
  `admin` tinyint(4) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `username_UNIQUE` (`username`),
  UNIQUE KEY `email_UNIQUE` (`email`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;
```
Edit index.py to allow remote access (assume port has been allowed to access):
```
Change the following line:

app.run_server(debug=True) -> app.run_server(host='0.0.0.0',debug=True)
```

Edit the file users_mgt.py:
```
password = db.Column(db.String(80)) -> password = db.Column(db.String(88))
```
Either you use Mysql command to create the first admin user account or Edit the file index.py to create admin access to create first admin user:
Original Code:
```python
    if pathname == '/admin':
        if current_user.is_authenticated:
            if current_user.admin == 1:
                return user_admin.layout
            else:
                return error.layout
        else:
            return login.layout
```
Modified code (temporary) to create admin user:
```python
    if pathname == '/admin':
        if True #if current_user.is_authenticated:
            if True: #if current_user.admin == 1:
                return user_admin.layout
            else:
                return error.layout
        else:
            return login.layout
```
Execute the python script:
```
 python3 index.py
 ```
Fire up browser and use the address http://localhost:8050/admin

Create the admin account and change back the code in index.py:
```python
    if pathname == '/admin':
        if current_user.is_authenticated:
            if current_user.admin == 1:
                return user_admin.layout
            else:
                return error.layout
        else:
            return login.layout
```
Goto main page to login http://localhost:8050

Add dash codes in views/page1.py and views/page2.php

## Modifications to views/page1.py to include an analysis of covid 19 data:
Hx has slighly modified some other codes from the web (https://towardsdatascience.com/studying-the-pandemic-with-a-single-visualization-using-plotly-dash-ae2a3431866c) and embed in page1.py. The resulting graph is shown above at the beginning of this page:
```python
# Dash packages
import dash_bootstrap_components as dbc
import dash_html_components as html

from app import app

###############################################################################
########### LANDING PAGE LAYOUT ###########
###############################################################################
'''
layout = dbc.Container([

        html.H2('Page 1 Layout'),
        html.Hr(),


], className="mt-4")
'''



import dash
from dash.dependencies import Output, Input
import dash_core_components as dcc
import dash_html_components as html
import plotly.express as px
import pandas as pd


#external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']
#app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

colors = {
    'background': '#FFFFFF',
    'text': '#7FDBFF'
}

# Reading The Dataset 

#data = pd.read_csv('https://gist.githubusercontent.com/ThiagoFPMR/fea32b8082a54889ba7470ac63252299/raw/aea145700257ffa89d924073189a0e3804bd987c/covid_worldwide.csv')
data = pd.read_csv('covid_worldwide.csv')

def scatter_y_label (var):
  if var == 'total_cases':
    return 'Percentage Infected'
  elif var == 'total_tests':
    return 'Percentage Tested'
  elif var == 'total_deaths':
    return 'Percentage Dead'
  elif var == 'total_recovered':
    return 'Percentage Recovered'

# Variable VS Education Level Scatter Plot

@app.callback(Output('covid-vs-edu', 'figure'),
              [Input('population-slider', 'value'),
               Input('interest-variable', 'value')])             
def update_scatter(selected_pop, interest_var):
  sorted = data[data.population <= selected_pop]
  fig = px.scatter(sorted,
                  x='expected_years_of_school',
                  y=sorted[interest_var]/sorted.population,
                  size='population',
                  color='income_group',
                  hover_name='country',
                  template='plotly_white',
                  labels={'expected_years_of_school':'Expected Years of School',
                          'y': scatter_y_label(interest_var)},
                  title='Total Cases VS Education Level')
  fig.update_layout(transition_duration=500)
  return fig

# Variable Per Income Group Bar Chart

@app.callback(Output('covid-vs-income', 'figure'),
              [Input('population-slider', 'value'),
               Input('interest-variable', 'value')])             
def update_income_bar(selected_pop, interest_var):
  sorted = data[data.population <= selected_pop].groupby(by='income_group').sum().reset_index()
  fig = px.bar(sorted,
                  x='income_group',
                  y=interest_var,
                  color='income_group',
                  template='plotly_white',
                  labels={'income_group':'Income Group',
                          'total_cases':'Total Cases',
                          'total_tests':'Total Tests',
                          'total_deaths':'Total Deaths',
                          'total_recovered':'Total Recovered'},
                  title='Total Cases By Income Group')
  fig.update_layout()
  return fig

# Variable Per Country Bar Chart

@app.callback(Output('covid-vs-income2', 'figure'),
              [Input('population-slider', 'value'),
               Input('interest-variable', 'value')])             
def update_country_bar(selected_pop, interest_var):
  sorted = data[data.population <= selected_pop]
  fig = px.bar(sorted, 
                x='country', 
                y=interest_var, 
                color='income_group',
                template='plotly_white',
                labels={'country':'Country',
                        'total_cases':'Total Cases',
                        'total_tests':'Total Tests',
                        'total_deaths':'Total Deaths',
                        'total_recovered':'Total Recovered'},
                title='Total Cases per Country')
  fig.update_layout()
  return fig



# Defining App Layout 

layout = html.Div(style={'backgroundColor': colors['background']}, children=[
      html.H1('Studying The Pandemic Worldwide', style={'textAlign':'center'}),
      html.Div([
          html.Div([ 
              html.Label('Population'),
              dcc.Slider(
                  id='population-slider',
                  min=data.population.min(),
                  max=data.population.max(),
                  marks={
                    72037 : '72K',
                    80000000 : '80M',
                    150000000 : '150M',
                    300000000 : '300M',
                    700000000 : '700M',
                    1000000000 : '1B',
                    1439323776 : '1.4B' 
                  },
                  value=data.population.min(),
                  step=100000000,
                  updatemode='drag'
              )
          ]),
          html.Div([
              html.Label('Interest Variable'),
              dcc.Dropdown(
                  id='interest-variable',
                  options=[{'label':'Total Cases', 'value':'total_cases'},
                           {'label': 'Total Tests', 'value':'total_tests'},
                           {'label': 'Total Deaths', 'value':'total_deaths'},
                           {'label': 'Total Recovered', 'value':'total_recovered'}],
                  value='total_cases' 
              )
          ])
      ], style = {'width':'90%','margin':'auto'}),      
      html.Div([ 
              dcc.Graph(
                  id='covid-vs-edu',
              ),    
          html.Div(
              dcc.Graph(
                  id='covid-vs-income',
              )
      , style = {'width': '50%', 'display': 'inline-block'}),
          html.Div( 
              dcc.Graph(
                  id='covid-vs-income2',
              )
      , style = {'width': '50%', 'display': 'inline-block'})
      ], style = {'width':'90%','margin':'auto'})
])
```


