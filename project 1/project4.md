#MEAN STACK DEPLOYMENT

after spinning up the server instance then...

Update ubuntu in the instance
`sudo apt update`

Upgrade ubuntu
`sudo apt upgrade`

Add certificates
`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

`curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`

##Install NodeJS
`sudo apt install -y nodejs`


##Step 2: Install MongoD
Run these commands

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

Now Install MongoDB
`sudo apt install -y mongodb`

Start The server
`sudo service mongodb start`

To verify that the service is up and running
`sudo systemctl status mongodb`

Once the server is running we can then Install npm – Node package manager.
`sudo apt install -y npm`

Install body-parser package to help us process JSON files passed in requests to the server.
`sudo npm install body-parser`

Create a folder named ‘Books’
`mkdir Books && cd Books`

In the Books directory, Initialize npm project
'npm init'
This will bring some input, we can keep pressing Enter on our keyboard till it's done.

Add a file to it named server.js
`vi server.js`

Copy and paste the web server code below into the server.js file.

`var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});`

