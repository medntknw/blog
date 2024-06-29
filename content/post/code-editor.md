+++
author = "Aniket Patanwal"
title = "Online code editor"
date = "2024-06-25"
description = "In this post we will create and deploy an online collaborative code editor"
summary = "This post we will see how we decide the requirements of the project and choose the tech stack. We will then develop our project and deploy"
draft=false
+++


## Where to start?

This is the hardest part of any project (for me at least  :p)
Lets list down the requirements
1. A webpage to write code on (This will be our client i.e browser)
2. Sync between all the clients connected to the webpage

For creating a webpage we can choose ReactJs which has multiple libraries which can help us with code editor part.

For the sync part we need a server which will receive edit requests and then broadcast the edit to all the connected client.
The server should also handle any conflicts in the data for which there are mainly two implementations
1. OT (Operational Transformations)
2. CRDT (Conflict-free Replicated Data Type)

Reference: https://nurkiewicz.com/2022/04/crdt.html#:~:text=The%20difference%20is%20that%20with,different%20because%20they%20are%20decentralized.

For implementing the server we can pick any language and framework. I selected Nodejs for its faster development speed and mature CRDT libraries like automerge, Yjs.


Finally,
We will use ReactJs for frontend (Editor)
and Nodejs for backend (websockets, http server)


## Setting up the development Environment

Before setting up lets understand the difference between npx and npm
npm -> (Node Package Manager)
npx -> (Node Package eXecute)
### Key Differences

1. **Installation**:
    
    - **npm**: Installs packages locally or globally, which are then available for use in your projects or globally on your system.
    - **npx**: Does not require installation of packages. It fetches and executes them on-the-fly from the npm registry or from a specified location.
2. **Usage Context**:
    
    - **npm**: Used for managing and installing packages, updating dependencies, and executing scripts defined in `package.json`.
    - **npx**: Used for running CLI tools and scripts directly without prior installation, helpful for occasional use or running specific package versions.
3. **Global vs. Local Scope**:
    
    - **npm**: Can install packages globally (`npm install -g <package>`) or locally within a project (`npm install <package>`).
    - **npx**: Executes packages without installing them globally or locally, ensuring that the command is executed with the latest version from the registry.

## Frontend setup

We will create our react app using

```
npx create-react-app codedoc-frontend
```


and then start the app by running

```
cd codedoc-frontend
npm start
```


## Creating the editor

We will be using the Monaco Editor and Yjs for utilising CRDT

Install using
```
npm install @monaco-editor/react
npm install yjs y-monaco y-websocket
```

Creating a component named Room which will have the Editor

/src/components/Room.js
```javascript
import Editor from '@monaco-editor/react';
function Room() {

	return (
		<Editor height="90vh" defaultLanguage="javascript" defaultValue="// some comment" />
	
	);

}
export default Room;
```

Your app will look like this
/src/App.js
```javascript
import './App.css';
import Room from './components/Room';

function App() {
	return (
		<div className="App">
			<Room/>
		</div>
		
	);

}
export default App;
```
![Example Image](/blog/images/editor.png)

## Constants

We will store all the constants variables like deafulat language, available languages and boilder plate code in constants.js

```javascript
export const LANGUAGES = [
    "javascript",
    "cpp",
    "python"
]

export const DEFAULT_LANGUAGE = "python"

export const LANGUAGE_COMMENT = {
    javascript: "// some comment",
    cpp: "// some comment",
    python: "# some comment"
}
```

## Language Selector

Lets create a component LanguageSelector.js will help change the language of the code editor

Lets improve our development speed by using pre built components ready to use

```
npm install @mui/material @emotion/react @emotion/styled
```

LanguageSelector.js
```javascript
const LanguageSelector = (props) => {

    const handleChange = (event) => {
        const {
            target: { value },
        } = event;
        props.onSelect(value);
    };

    return (
        <FormControl sx={{ width: 300, mt:3, mb: 1}}>
            <InputLabel>Language</InputLabel>
            <Select
                value={props.selected}
                onChange={handleChange}
                input={<OutlinedInput label="Language" />}
                MenuProps={MenuProps}
            >
            {LANGUAGES.map((language) => (
                <MenuItem
                key={language}
                value={language}
                >
                {language}
                </MenuItem>
            ))}
            </Select>
        </FormControl>
    )
}

export default LanguageSelector
```



## Setting up backend
As we decided we will be running our backend on express using Node.js

```
mkdir codedoc-backend
cd codedoc-backend
npm init -y
npm install express ws yjs y-websocket
vim server.js
```


server.js

```javascript
const express = require('express');

const http = require('http');

const WebSocket = require('ws');

const { setupWSConnection } = require('y-websocket/bin/utils');

  

const app = express();

app.use(express.json());

const server = http.createServer(app);

const wss = new WebSocket.Server({ server });


wss.on('connection', (ws, req) => {

	setupWSConnection(ws, req);

});
server.listen(1234, () => {

	console.log('Server is listening on port 1234');

});
```

Here the setupWSConnection handles the broadcasting of the messages to all connected clients

## Binding the Monaco Editor with Yjs
This method is used for creating shareable data structure Y.doc and setup websocket connection to our nodejs backend

```javascript
const syncEditor = (editor) => {
	const doc = new Y.Doc();
	const newProvider = new WebsocketProvider('ws://localhost:1234', roomId, doc);

	const ytext = doc.getText("monaco");
	
	const newBinding = new MonacoBinding(ytext, editor.getModel(), new Set([editor]), newProvider.awareness);
	
	});
	setProvider(newProvider);
	setBinding(newBinding);

};
```
## Fast Forwarding...

Now what I did was basically set beautify the react app and made some nit fixes.
You can check the complete code here
https://github.com/medntknw/codedoc

The editor is also online for testing but the backend sometimes becomes unresponsive (have deployed it on render.com) :(\
https://codedoc-frontend.vercel.app/



