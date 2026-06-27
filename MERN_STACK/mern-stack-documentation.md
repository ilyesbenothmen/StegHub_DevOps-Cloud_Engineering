## MERN Web Stack Implementation Documentation
### Overview
This project documents the implementation of a MERN web stack environment for a Todo application hosted on an Ubuntu-based AWS EC2 instance. 

The MERN stack combines MongoDB, Express.js, React, and Node.js to support full-stack JavaScript development. In this lab, the documented steps focus on preparing the server and backend workspace as the foundation for building and deploying the application,after that we will validate the REST api with postman and develop a React code to build finally a complete application

### Objective
The objective of this lab was to prepare a working MERN stack development environment on a remote Linux server and begin the setup of a Todo application project. This included connecting securely to the EC2 instance, updating system packages, installing Node.js and npm, confirming the installed versions, and initializing the Node.js project directory,configuring a mongodb as a managed service and validate the backend with Postman then developping a React code to consume the REST api.

### Technologies Used
AWS EC2 instance running Ubuntu 26.04 LTS.

Linux terminal commands for server administration.

Node.js runtime environment.

npm package manager.

NodeSource repository for installing the current Node.js release.

MongoDB Atlas as the cloud-hosted database service used by the application.

React user interface for interacting with the REST API from the frontend.

REST API communication between the frontend and backend application layers.

### Step0 preparing prerequisites:
First of all , we are going to spin up an EC2 instance running on AWS.
![alt](./images/1.png)
After that we need to implement inbound rules to allow connection from the internet
![alt](./images/2.png)

The implementation began with secure access to the remote Ubuntu EC2 instance using an SSH private key. File permissions were first adjusted on the .pem key file, after which the connection to the server was established successfully.

```bash
chmod 600 steghub-mern-project3.pem
ssh -i steghub-mern-project3.pem ubuntu@54.91.219.145
```
![alt](./images/3.png)

### Step1 Backend configuration

After logging in, the next task was to refresh the package index so that the server could fetch the latest package metadata from Ubuntu repositories.

```bash
sudo apt update
```
This step ensured that subsequent package installation commands would use updated repository information.
![alt](./images/4.png)

Once the package index was refreshed, the system packages were upgraded. This is a good practice before software installation because it reduces compatibility issues and brings the server to a more current state.

```bash
sudo apt upgrade
```
![alt](./images/5.png)
The terminal output shows that several standard LTS security updates were available and prepared for installation.

To install a modern Node.js version, the NodeSource setup script was executed. This step configured the external repository and prepared the server to install Node.js version 22.x.

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
```
![alt](./images/6.png)
The output confirms that prerequisites were installed and that the NodeSource repository was configured successfully.
![alt](./images/7.png)
Now, the system is ready to install Node.js.

Let's install nodejs with the following command
```bash
sudo apt-get install -y nodejs
```
![alt](./images/8.png)
With the repository configured, Node.js was installed using the package manager.

After installation, the versions of Node.js and npm were checked to verify that both tools were available and working correctly in the environment.

```bash
node -v
npm -v
```
![alt](./images/9.png)
The screenshots show that Node.js version v22.23.0 and npm version 10.9.8 were installed successfully.

Application Code Setup :
The next stage was to prepare a workspace for the Todo application. A directory named Todo was created and opened as the working project folder.

```bash
mkdir todo
cd Todo/
```
![alt](./images/10.png)
This step established the base directory where the application files and dependencies would be stored.

Initialize the Node.js Application
Inside the project directory, the Node.js application was initialized using npm init. This command generated the package.json file that defines the application metadata and dependency configuration.

```bash
npm init
```
![alt](./images/11.png)
During initialization, the default project values were accepted, including the package name todo, version 1.0.0, and entry point index.js.

The generated package.json file shows the basic project structure created by npm. This file serves as the foundation for adding dependencies and scripts in the next stages of the MERN application setup.

![alt](images/12.png)
![alt](images/13.png)
Express is a framework for Node.js , so let us install it:
```bash
npm install express
```
![alt](images/14.png)

After that create install dotenv module with
```bash
npm install dotenv
```
![alt](images/15.png)

After that type the following js code:

```JS
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use((req, res, next) => {
  res.send('Welcome to Express');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```
![alt](./images/16.png)
Now it is time to run our first server program !
```bash
node index.js
```
![alt](./images/17.png)
port 5000 , should be open  in ec2 security group.

![alt](./images/18.png)

Now we need to test our first dummy application , Open the browser and enter the public dns name and port 5000

 ![alt](./images/19.png)

 Our program is working !



**Routes**
There are three actions that the ToDo application needs to be able to do:

1. Create a new task
2. Display list of all task
3. Delete a completed task

Each task was associated with some particular endpoint and used different standard HTTP request methods: POST, GET, DELETE.
First of all , let us create routes directory inside which we create an api.js file

 After that ,copy the code bellow in the file api.js

 ```JS
 const express = require('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

});

module.exports = router;
```
![alt](images/20.png)

**Models**
A model is at the heart of JavaScript-based applications and is what makes them interactive.

Models are used to define the database schema. This is important for specifying the fields stored in each MongoDB document.

In essence, a schema is a blueprint of how the database is structured, including additional fields that may not need to be stored in the database. These are known as virtual properties.

To create schemas and models, Mongoose is used, a Node.js package that simplifies working with MongoDB.
Change directory to Todo directory and install mongoose with
```js
npm install mongoose
```
![alt](./images/21.png)
after that create a directory named models inside which create a todo.js script
```bash
mkdir models && cd models && touch todo.js
```
Than write the todo.js script , like the following
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
});

// Create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
![alt](./images/22.png)

Now the file routes/api.js should be modified like the following:
```js
const express = require('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {
  // This will return all the data, exposing only the id and action field to the client
  Todo.find({}, 'action')
    .then(data => res.json(data))
    .catch(next);
});

router.post('/todos', (req, res, next) => {
  if (req.body.action) {
    Todo.create(req.body)
      .then(data => res.json(data))
      .catch(next);
  } else {
    res.json({
      error: "The input field is empty"
    });
  }
});

router.delete('/todos/:id', (req, res, next) => {
  Todo.findOneAndDelete({"_id": req.params.id})
    .then(data => res.json(data))
    .catch(next);
});

module.exports = router;
```
![alt](./images/23.png)

Now we are done with routes and we will move to a DBaaS Mongodb Atlas

**Mongo Database**

**mLab** provides a MongoDB managed service (DBaaS), which simplifies database management — especially useful in this lab since the infrastructure is hosted on **AWS** EC2.
First , I created a free account on cloud.mongodb.com

![alt](./images/24.png)
then create a project named Project 0

![alt](./images/25.png)
After that create a cluster

![alt](./images/26.png)
Specify that our cluster will be a free tier

![alt](./images/27.png)
Specify it is an **AWS** provider

![alt](./images/28.png)
Then create a db username and a password

![alt](./images/29.png)
Then choose a connection method

![alt](./images/30.png)

After that specify Driver as a choice 

![alt](./images/31.png)

Then select Node.js version 6.7 or later and note the connection string that will be used later.

![alt](./images/32.png)
Then specify an Access List entry to filter how can connect

![alt](./images/33.png)

After that a database named **todo-db** and a collection named todo 
![alt](./images/34.png)
Now our database is ready and we are going to configure our environment to access database.
For that reason create under Todo an environment file named .env with credentials to access database.

![alt](./images/35-1.png)
now create an index.js file with the following code 

```js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

// Connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log(`Database connected successfully`))
  .catch(err => console.log(err));

// Since mongoose promise is deprecated, we override it with Node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
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
  console.log(`Server running on port ${port}`);
});

```

![alt](./images/36.png)
All you have now , is to start index.js with

```js
node index.js
```

![alt](./images/37.png)
**Testing Backend Code without Frontend using RESTful API**
To test a REST API, we need a tool that provides a simple GUI, such as Postman.
to test we get the public IP like 100.27.209.182
```html
GET http://100.27.209.182:5000/api/
```

![alt](./images/38.png)

Choose the method **POST** and Specify in Headers Content-Type = application/json

![alt](./images/39.png)
Choose body raw and run the request , we get our first JSON result set.

![alt](./images/40.png)
Insert some raws with our POST request

![alt](./images/41.png)
Now choose **DELETE** method to clean the todo list

![alt](./images/42.png)
Insert a task 

![alt](./images/43.png)
Now verify the todo list

![alt](./images/44.png)


![alt](./images/45.png)


![alt](./images/46.png)
Our REST API has now been validated.
**Step 2 Frontend creation**

we need to scaffold our application , with the command below we are going to generate a skeleton for our web client.
```bash
npx create-react-app client

```

![alt](./images/47.png)


![alt](./images/48.png)

It is recommended to install concurrently to run more than one command simultanousely from the same  terminal ,  install also nodemon to monitor the server, if there is any change in the server code , nodemon will restart it automatically nad load the new changes.
```bash
Our REST API has now been validated.
npm install concurrently --save-dev
npm install nodemon --save-dev
```
![alt](./images/49.png)
- In Todo folder open the package.json file, change the highlighted part of the below screenshot and replace with the code below:
```JS
"scripts": {
  "start": "node index.js",
  "start-watch": "nodemon index.js",
  "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
}
```

![alt](./images/50.png)

**Configure  Proxy in package.json**
move to Todo/client and add "proxy": "http://localhost:5000", in package.json
![alt](./images/51.png)


Now inside the Todo directory run 
```bash
npm run dev
```
![alt](./images/54.png)

>[!Note]
> To allow access to the service on port 3000 ensure to allow connection from outside in the inbound rule

**Creating React Components**
The importance of using components is that they make the application modular and reusable, as recommended in the SDLC.

Now create a directory components under Todo/client/src/
```bash
mkdir -p client/src/components
````
inside components create Input.js ListTodo.js and Todo.js
```bash
touch Input.js ListTodo.js Todo.js
```
inside Input.js Copy and paste the following:
```JS
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {
  state = {
    action: ""
  }

  handleChange = (event) => {
    this.setState({ action: event.target.value });
  }

  addTodo = () => {
    const task = { action: this.state.action };

    if (task.action && task.action.length > 0) {
      axios.post('/api/todos', task)
        .then(res => {
          if (res.data) {
            this.props.getTodos();
            this.setState({ action: "" });
          }
        })
        .catch(err => console.log(err));
    } else {
      console.log('Input field required');
    }
  }

  render() {
    let { action } = this.state;
    return (
      <div>
        <input type="text" onChange={this.handleChange} value={action} />
        <button onClick={this.addTodo}>add todo</button>
      </div>
    );
  }
}

export default Input;
```
After that open the ListTodo.js
```bash
vim ListTodo.js
```
Copy and paste the following code:
```js
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {
        todos && todos.length > 0 ? (
          todos.map(todo => {
            return (
              <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
                {todo.action}
              </li>
            );
          })
        ) : (
          <li>No todo(s) left</li>
        )
      }
    </ul>
  );
}

export default ListTodo;

```

Then in the Todo.js file, write the following code

vim Todo.js

```bash
vim Todo.js
```
Copy and paste the following code
```js
import React, { Component } from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {
  state = {
    todos: []
  }

  componentDidMount() {
    this.getTodos();
  }

  getTodos = () => {
    axios.get('/api/todos')
      .then(res => {
        if (res.data) {
          this.setState({
            todos: res.data
          });
        }
      })
      .catch(err => console.log(err));
  }

  deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if (res.data) {
          this.getTodos();
        }
      })
      .catch(err => console.log(err));
  }

  render() {
    let { todos } = this.state;
    return (
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos} />
        <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
      </div>
    );
  }
}

export default Todo;

```
We need to make a little adjustment to our react code. Delete the logo and adjust our App.js to look like this
Move to src folder and open App.js 
Copy and paste the following
```js
import React from 'react';
import Todo from './components/Todo';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <Todo />
    </div>
  );
}

export default App;
```
In the src directory, open the App.css
vim App.css
```css
.App {
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
    width: 100%;
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
}

```
In the src directory, open the index.css
```css
vim index.css
```
```js
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  background-color: #282c34;
  color: #787a80;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New", monospace;
}
```
Go to the Todo directory and run again 
```bash
npm run dev
```
![alt](./images/54.png)

At this point, the ToDo app is ready and fully functional with the functionality discussed earlier: Creating a task, deleting a task, and viewing all the tasks.

![alt](./images/55.png)

![alt](./images/56.png)

![alt](./images/57.png)

**Conclusion**
By following the steps outlined in this lab, we developed a fully functional MERN stack application, illustrating the design and implementation of a complete full-stack web application.



