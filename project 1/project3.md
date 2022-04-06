# PROJECT 3: MERN STACK IMPLEMENTATION


## Step 1 – backend configuration

###Update ubuntu
After provisioning the EC2 instance and gaining access to the terminal...
`sudo apt update`
![Update](https://github.com/femie15/darey/blob/main/project%201/project3/1.PNG)

###Upgrade ubuntu
`sudo apt ugrade`
!(https://github.com/femie15/darey/blob/main/project%201/project3/2-upgrade-apt.PNG)


Getting Nodejs location from Ubuntu repositories
`curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`

Install  Nodejs and NPM
`sudo apt-get install -y nodejs`

check version of nodeJs and NPM with the command below
`node -v` for node and  `npm -v` for NPM

Create a new directory for your To-Do project:
`mkdir Todo`

Verify that the Todo directory is created with ls command
`ls`

Getting into the Todo directory
`cd Todo`

initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.

`npm init`

###INSTALL EXPRESSJS (framework for nodejs)

To use express, install it using npm:

`npm install express`

Now create a file index.js 

`touch index.js`

Run `ls` to confirm that index.js file is successfully created

Then Install the dotenv module

`npm install dotenv`

Open the index.js file with the command below

`nano index.js` or `vim index.js`

paste the code below

`const express = require('express');
require('dotenv').config();
const app = express();
const port = process.env.PORT || 5000;
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
app.use((req, res, next) => {
res.send('Welcome to Express');
});
app.listen(port, () => {
console.log(\``Server running on port ${port}\``)
});`


Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type:

`node index.js`

We now need to edit the outbound of our EC2 Security group to listen to the port 5000 and then view the welcome page from our browser using the server url anad port number.

`http://<PublicIP-or-PublicDNS>:5000`

we can get our PublicIP 
`curl -s http://169.254.169.254/latest/meta-data/public-ipv4` 

or PublicDNS using these curl command below
`curl -s http://169.254.169.254/latest/meta-data/public-hostname`

Our To-Do application needs to be able to:

1. Create a new task
2. Display list of all tasks
3. Delete a completed task

##Routes

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes

`mkdir routes`

Change directory to routes folder.

`cd routes`

Now, create a file api.js with the command below

`touch api.js`

Open the file with the command below

`vim api.js` or `nano api.js`

Copy below code in the file. (Do not be overwhelmed with the code)

`const express = require ('express');
const router = express.Router();
router.get('/todos', (req, res, next) => {
});
router.post('/todos', (req, res, next) => {
});
router.delete('/todos/:id', (req, res, next) => {
})
module.exports = router;`


##MODELS

Change directory back Todo folder with `cd ..` and install Mongoose

`npm install mongoose`

Create a new folder models :
`mkdir models`

Change directory into the newly created ‘models’ folder with
`cd models`

Inside the models folder, create a file and name it todo.js
`touch todo.js`

All three commands above can be defined in one line to be executed consequently with help of && operator, like this:

`mkdir models && cd models && touch todo.js`


Open the file created with vim todo.js then paste the code below in the file:

`const mongoose = require('mongoose');
const Schema = mongoose.Schema;
//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})
//create model for todo
const Todo = mongoose.model('todo', TodoSchema);
module.exports = Todo;`


Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.

In Routes directory, open api.js with `vim api.js`, delete the code inside with `:%d` command and paste there code below into it then save and exit

`const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');
router.get('/todos', (req, res, next) => {
//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});
router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});
router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})
module.exports = router;`


###Creation of MongoDB Database

Use MLab cloud service to provision the db. 'https://www.mongodb.com/atlas-signup-from-mlab' ensure you allow access from anywhere in the network access, create own DB from collection and select "Add my own Data".

You can now go back to the Todo directory and create a ".env" file. `touch .env`

and paste the Mongo DB credentials in it this way `DB = mongodb+srv://<username>:<password>@todo.tknie.mongodb.net/myFirstDatabase?retryWrites=true&w=majority` note that "todo.tknie.mongodb.net" is the nework address and 'myFirstDatabase" is the DBname

Go back to the Todo directory and update the index.js file with the code below

Begining of code

`const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});`

End of code

then restart the node server with `node index.js` if the server is complaining about an already running server then enter this to get the running processes ID that relates to node `ps aux | grep node` then get the current running node ID and use this code to terminate the current process `kill <Process ID>` you can now perform `node index.js`

##Testing Backend Code without Frontend using RESTful API

We can use postman to test the endpoints http://<ipaddress>:5000/api/todos with the various http requests "POST, GET, DELETE"
the post requests has a sample payload of 
  '{
      "action":"The first payload"
  }'
  

## Step 2 – frontend creation
  
we install react in the Todo directory `npx create-react-app client`
  
we need to also install concurrently to enable many command run simultanously in the terminal.
  `npm install concurrently --save-dev`
  
We also install nodemone to monitor the server and autochange when there's a code change in the server.
  `npm install nodemon --save-dev`
  
In Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace the script section the code below.
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
  

###Configure Proxy in package.json
  
Change directory to ‘client’
`cd client`
Open the package.json file
`vi package.json`
Add the key value pair in the package.json file "proxy": "http://localhost:5000".
The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

Now, ensure you are inside the Todo directory, and simply do:

`npm run dev`
  
  
From your Todo directory run

`cd client`
move to the src directory

`cd src`
Inside your src folder create another folder called components

`mkdir components`
Move into the components directory with

`cd components`
Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js

`touch Input.js ListTodo.js Todo.js`
Open Input.js file

`vi Input.js`
Copy and paste the following

`import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input`
  
  
To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run `yarn add axios` or npm install axios

Move to the src folder and Move to clients folder

`cd ../../`


Install Axios

`npm install axios`
  
  
##Step 2 – Frontend creation

 Go to ‘components’ directory

`cd src/components`
After that open your ListTodo.js

`vi ListTodo.js`
in the ListTodo.js copy and paste the following code

`import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo`
  
Then in your Todo.js file you write the following code

`import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;`
  
We need to make little adjustment to our react code. Delete the logo and adjust our App.js to look like this.

Move to the src folder

`cd ..`
Make sure that you are in the src folder and run

`vi App.js`
Copy and paste the code below into it

`import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;`
  
After pasting, exit the editor.

In the src directory open the App.css

`vi App.css`
Then paste the following code into App.css:

`.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}`
Exit

In the src directory open the index.css

`vim index.css`
Copy and paste the code below:

`body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}`
Go to the Todo directory

`cd ../..`
When you are in the Todo directory run:

`npm run dev
  
  ##DONE
