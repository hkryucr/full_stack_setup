### MongoDB
### Express
### React
### Node

### Useful VScode extensions
- Atom Keymap (to keep your familiar keyboard shortcuts)
- ES7 React/Redux/GraphQL/React-Native snippets (provides shortcuts for creating React components)
- Prettier (auto-formatter)
- Live Server (Launch a development local server with live reload features)
- Node.js Modules Intellisense (Autocompletes Node.js modules in import statements)


### MongoDB Atlas

- Go to the website and create an account
- Click Build a cluster
- Select Amazon Web Services as your cloud provider
- Select the region that geographically closest to you
- Leave everything else as defaults
- Wait for the cluster to be created (7-10 min)
- Click the Connect button on the newly created cluster
- Click Add a different IP address and enter 0.0.0.0/0 to allow connections from any IP
- Next create a new user
- Give them whatever username you like I.E. dev Click the Autogenerate Secure Password button and save the password for later - DO NOT COMMIT THE PASSWORD
- Click chose a connection method
- Click Connect Your Application
- Copy and paste the connection string
- replace the <username> with the username we just created
- replace the <password> with the auto-generated password
- now that we have this connection string we want to save it somewhere it won’t be committed




*******

### initial setting
- `npm init`
- `npm install express mongoose passport passport-jwt jsonwebtoken body-parser bcryptjs validator `
- `npm install -D nodemon`

### Creating a server
- create app.js
- in app.js, write a skeleton code to set up the server
```
const express = require('express');
const app = express();

// Setup the path
app.get("/", (req, res)=>{
    res.send("Hello World!");
})

// Check the environmental variable port, if it exist, use it. Otherwise, use 5000 
const port = process.env.PORT || 5000;

app.listen(port, () => {
    console.log(`Listening on port ${port}`);
})
```
- run node app.js to check if the server is running
- change the package.json file
```{
  "name": "mern-twitter-from-scratch",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "server:debug": "nodemon --inspect app.js",
    "server": "nodemon app.js",
    "start": "node app.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "body-parser": "^1.19.0",
    "express": "^4.17.1",
    "jsonwebtoken": "^8.5.1",
    "mongoose": "^5.9.4",
    "passport": "^0.4.1",
    "passport-jwt": "^4.0.0",
    "validator": "^12.2.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.2"
  }
}

```

### Connect node with mongoDB
- run `git init`
- create `.gitignore`
- in .gitignore, write as follows
```
/node_modules
keys.js

```

- create config folder and create keys.js in the config folder
- in keys.js file
```
module.exports = {
  mongoURI:
    "mongodb+srv://<name>:<random generated secure password>@cluster0-cmn44.mongodb.net/test?retryWrites=true&w=majority"
};
```

- in app.js file, add these lines
```
const mongoose = require("mongoose");
const db = require("./config/keys").mongoURI;

mongoose
  .connect(db, { useUnifiedTopology: true })
  .then(() => {
    console.log("Connected to mongoDB");
  })
  .catch(err => console.log(err));
```

### Create routes in app.js and route files


- create route files
```
const express = require("express");
const router = express.Router();

router.get("/test", (req, res) => {
  res.json({ msg: "This is the tweet route" });
});

module.exports = router;
```

- in app.js, import route files and call it by using app.use("route that you want to make", routefile)

```
const users = require("./routes/api/users");
const tweets = require("./routes/api/tweets");

app.use("/api/users", users);
app.use("/api/tweets", users);

```
- the real route would be "localhost:5000/api/tweets/test"


### User Auth

- Create a User file (User.js) in models folder
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
    handle: {
        type: String,
        required: true
    },
    email: {
        type: String,
        required: true
    },
    password: {
        type: String,
        required: true
    },
    date: {
        type: Date,
        default: Date.now
    }
});

const User = mongoose.model('users', UserSchema);

module.exports = User;
```

- in app.js, we can create a new user instance as follows
```
const User = require('./models/User');

const user = new User({
        handle: "jim",
        email: "jim@aa.io",
        password: "jimisgreat123"
    });
    user.save();
```

- if we want other http requests, we could create a new type of request with a new route as follows:

```
router.post('/register', (req, res)=>{
    User.findOne({
        email: req.body.email
    }).then(user => {
        if(user){
            return res.status(400).json({email: "A user is already registered with that email"})
        } else {
            const newUser = new User({
                handle: req.body.handle,
                email: req.body.email,
                password: req.body.password
            })

            bcrypt.genSalt(10, (err, salt) => {
                bcrypt.hash(newUser.password, salt, ( err, hash ) => {
                    if(err) throw err;
                    newUser.password = hash;
                    newUser.save()
                    .then((user)=>res.json(user))
                    .catch(err => console.log(err));                    
                })
            })


            newUser.save().then(user=>res.send(user)).catch(err => res.send(err));
        }
    });
})
```
