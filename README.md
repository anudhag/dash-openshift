# Dash on OpenShift

## Overview of Dash
Dash is a productive Python framework for building web applications. While Dash 
apps are viewed in the web browser, you don’t have to write any Javascript or 
HTML. Dash provides a Python interface to a rich set of interactive web-based
 components. Dash is an open source library, released under the permissive MIT
 license.  Plotly develops Dash and offers a platform for deploying,
 orchestrating, and permissioning dash apps in an enterprise environment.

### High level features:

- Requires minimal lines of code.
- Generated entirely from Python, even the HTML and JS.
- Binds interactive components (dropdowns, graphs, sliders, text inputs) with
 Python code through "callbacks".
- Apps are "reactive" which means we can have multiple inputs, multiple outputs,
 and inputs that depend on other inputs.
- Inherently multi-user apps
- Plugin system for creating your own Dash components with React.
- Dash's Graph component is interactive hence responds to hovering, clicking, or
 selecting points on the graph.

## Dash on OpenShift

Deploying Dash apps on OpenShift has numerous benefits. A few of which are:

##### Application Portability:

Built around a standardized container model powered by docker. Which means that
 any application created on OpenShift can easily run anywhere that supports
 docker.

##### Scalable:
Enables applications to be scaled easily to handle increased traffic and demand
 on the applications.

##### Multiple Interaction Models:
Developers can create and manage applications utilizing a rich set of
 command-line tools, a powerful multi-device web console, or an Eclipse-based
 Integrated Development Environment such as JBoss Developer Studio.

##### Persistence:
OpenShift allows platform architects the choice to incorporate persistence into
 their application component while still be able to offer stateless cloud native
 design.

##### Choice of Cloud Infrastructure:
OpenShift provides customers with the choice to run on top of physical or 
virtual, public or private cloud, and even hybrid cloud infrastructure. 
This gives IT the freedom to deploy Red Hat OpenShift Container Platform in a
 way that best fits within the existing infrastructure.

##### Open Source:
Which means true transparency instead of some of those "Open Core" choices out
 there. It's not just with source code either, that includes best practices.

## Getting Started
Lets start with creating and deploying a basic Dash app. We'll need python, git,
 virtualenv and an OpenShift account. The following eample is built using
 python3 on a RHEL machine.

```bash
$ mkdir dash_openshift
$ cd dash_openshift
$ virtualenv venv -p python3
$ source venv/bin/activate
$ git init
```

### Installing packages

Install the required libraries as follows:

```bash
$ pip install dash==0.21.0
$ pip install dash-core-components==0.21.0
$ pip install dash-html-components==0.9.0
$ pip install dash-renderer==0.11.3
```
## Building Dash App

Now we are all set to build our first Dash application.
Create a file `wsgi.py` with the following content:

```python
# -*- coding: utf-8 -*-
import dash
import dash_core_components as dcc
import dash_html_components as html

app = dash.Dash()
application = app.server

app.layout = html.Div(children=[
    html.H1(children='Hello Dash'),

    html.Div(children='''
        Dash: A web application framework for Python.
    '''),

    dcc.Graph(
        id='example-graph',
        figure={
            'data': [
                {'x': [1, 2, 3], 'y': [4, 1, 2], 'type': 'bar', 'name': 'SF'},
                {'x': [1, 2, 3], 'y': [2, 4, 5], 'type': 'bar', 'name': u'Montréal'},
            ],
            'layout': {
                'title': 'Dash Data Visualization'
            }
        }
    )
])

app.css.append_css({"external_url": "https://codepen.io/chriddyp/pen/bWLwgP.css"})

if __name__ == '__main__':
    app.run_server(port=8080, debug=True)

```
This is available on [Github](https://github.com/anudhag/dash-openshift.git)

### Running the app locally
Run the app locally

```bash
$ python wsgi.py 
```
The app can be accessed at http://localhost:8080/

### Deploying to OpenShift
We need a few more steps and our app is ready to be deployed to OpenShift.  
First we'll need to add a 
[WSGI server](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) for 
running the app on OpenShift. We'll use [gunicron](http://gunicorn.org/) here.  

```bash
$ pip install gunicorn
```

We'll add the gunicorn configurations in a config.py file. Add a `config.py` file with the following content:  

```python
import os

workers = int(os.environ.get('GUNICORN_PROCESSES', '3'))
threads = int(os.environ.get('GUNICORN_THREADS', '1'))

forwarded_allow_ips = '*'
secure_scheme_headers = { 'X-Forwarded-Proto': 'https' }
```

Now we need to set the environment variable `APP_CONFIG` to point ot this config file, for that 
we'll need to add an [environment](https://docs.openshift.org/latest/dev_guide/builds/build_strategies.html#environment-files) 
file within a `.s2i` directory.

```bash
$ mkdir .s2i
$ cd .s2i
```

Add the following content to an `environment` file in `.s2i` directory.

```
APP_CONFIG=config.py
```

Create a `requirements.txt` file in the `dash_openshift` directory with a list of packages to be installed.  
More info about requirements file can be found 
[here](http://pip-python3.readthedocs.io/en/latest/user_guide.html#requirements-files)

```bash
$ cd ..
$ pip freeze > requirements.txt
```

Now we are all set for deployment. Push the code to GitHub.

```bash
$ echo venv/ > .gitignore
$ git add .
$ git commit -m "First commit"
$ git remote add origin <remote repo url>
$ git push origin master
```

<br>
Login to you OpenShift Online console. Click '+ Create Project', add project 
details and click 'Create'.  
<br>
<br>

![Create Project](/blog_images/create_project.png)

<br>
<br>
Click on the newly created project and 'Browse Catalog', select 'Python' and 
click 'Next'.  
<br>
<br>

![Browse Catalog](/blog_images/browse_catalog.png)

<br>
<br>
Choose Python version, enter application name, add git repositiry url and click 
'Create'.  
<br>
<br>

![Add Python Image](/blog_images/add_python_image.png)

<br>
<br>

Once the project is created, close the dialog and go to 'Overview', where 
you'll see the project is being built and then deployed.  
Once deployment is complete you'll be able to open your app (similar to 
[this](http://demo-dash.7e14.starter-us-west-2.openshiftapps.com))

### Adding some interactivity
Now lets make our app more interactive.
Lets add a graph with a dropdown and slider to update the graph.  
Consider 
[this](https://github.com/anudhag/dash_openshift_example/blob/master/wsgi.py) 
dash application.  
Update your wsgi.py and run the app locally. It'll look like 
[this](http://demo-dash2.7e14.starter-us-west-2.openshiftapps.com/)

The interactivity in this app is added using a callback function.  

```python
@app.callback(Output('my-graph', 'figure'),
             [Input('my-dropdown', 'value'),
              Input('my-slider', 'value')])
def update_graph(dropdown_vals, slider_val):
    ...
```

A callback function uses the `app.callback` decorator and specifies the inputs 
to read and the output to update. In this case the `value` of the dropdown and 
the slider are the inputs and the output is the `figure` object of the graph.


### Style the application with CSS
Custom styling can be added to the app components using the style argument, 
providing the custom css in a dictonary format. For instance:

```python
html.Div([],
         style={'backgroundColor': 'rgb(56, 83, 108)',
                'color':'white', 'padding': 30})
```

Commit your changes and push to GitHub. In order to update your app on 
OpenShift click 'Start Build' from the Overview page of the OpenShift web 
console.

![Build Project](/blog_images/start_build.png)

### Getting pandas to work
In case you are using Pandas in you application and your build fails with 
'OOM Killed' message, try to increase the 
[build resource limit](https://docs.openshift.com/container-platform/3.7/dev_guide/builds/advanced_build_operations.html#build-resources) 
by adding the following in your Build Config yaml file:

```yaml
resources:
 limits:
  memory: 1Gi
```

Now you know how simple it is to create a Dashboard using Dash and OpenShift.
You can learn more about Dash [here](https://dash.plot.ly/getting-started)

