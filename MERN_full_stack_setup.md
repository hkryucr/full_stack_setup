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
