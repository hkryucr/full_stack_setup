# Fullstack with Rails, React, and Redux

- Create backend with Rails
```
rails _5.2.3_ new my_project_name -G --database=postgresql --skip-turbolinks
```

- Update Gemfile
```
gem 'better_errors'
gem 'binding_of_caller'
gem 'pry-rails'
gem 'annotate'
gem 'bcrypt'
gem 'jquery-rails'
``` 

- run `bundle install`

- update application.js file
```
//= require jquery
//= require rails-ujs
```

- run 'git init' and create .gitignore file and write these codes to prevent from pushing unwanted files
```
node_modules/
bundle.js
bundle.js.map

# TODO Comment out this rule if you are OK with secrets being uploaded to the repo
# config/initializers/secret_token.rb
config/master.key
config/initializers/secret_token.rb

# Ignore all logfiles and tempfiles.
/log/*
/tmp/*
!/log/.keep
!/tmp/.keep

## Environment normalization:
/.bundle
/vendor/bundle


# Ignore uploaded files in development
/storage/*
!/storage/.keep

# Ignore Byebug command history file.
.byebug_history

# Ignore node_modules
/public/packs
/public/packs-test
/public/assets
/tmp


```

- create a package.json file by running `npm init --yes` and create a folder called `frontend` including `actions`, `reducers`, `store`, `util`, and `components` folders

- install frontend frameworks and libraries
```
npm install --save webpack webpack-cli react react-dom react-router-dom redux react-redux @babel/core @babel/preset-react @babel/preset-env babel-loader lodash redux-logger redux-thunk
```
- optionally, if you want to use css-loader, install as follows, which enables to use `import './another-stylesheet.css'` inside js/jsx files.
```
npm install --save-dev css-loader
npm install --save-dev style-loader

```
- in package.json, add the following code under the scripts `"start": "webpack --watch --mode=development"`
```
{
  "name": "yocal",
  "private": true,
  "dependencies": {
    "@babel/core": "^7.8.6",
    "@babel/preset-env": "^7.8.6",
    "@babel/preset-react": "^7.8.3",
    "babel-loader": "^8.0.6",
    "lodash": "^4.17.15",
    "react": "^16.13.0",
    "react-dom": "^16.13.0",
    "react-redux": "^7.2.0",
    "react-router-dom": "^5.1.2",
    "redux": "^4.0.5",
    "redux-logger": "^3.0.6",
    "redux-thunk": "^2.3.0",
    "webpack": "^4.41.6",
    "webpack-cli": "^3.3.11"
  },
  "devDependencies": {
    "css-loader": "^3.4.2"
  },
  "description": "This README would normally document whatever steps are necessary to get the application up and running.",
  "version": "1.0.0",
  "main": "sliceOfState.js",
  "directories": {
    "lib": "lib",
    "test": "test"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack --watch --mode=development"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/hkryucr/yocal.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/hkryucr/yocal/issues"
  },
  "homepage": "https://github.com/hkryucr/yocal#readme"
}

```

- create `frontend` folder and inside it, make `reducers, util, actions, store, components` folder, and `entry_file_name.jsx`.

- create `webpack.config.js` file and set up the entrypoint, output path, and babel transpilation, set up the absolute paths, and include devtool: `source-map`
```
const path = require("path");

module.exports = {
    context: __dirname,
    entry: "./frontend/yocal.jsx",
    output: {
        path: path.resolve(__dirname, "app", "assets", "javascripts"),
        filename: "bundle.js"
    },
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader'],
            },
            {
                test: /\.jsx?$/,
                exclude: /(node_modules)/,
                use: {
                    loader: "babel-loader",
                    query: {
                        presets: ["@babel/env", "@babel/react"]
                    }
                }
            }
        ]
    },
    devtool: "source-map",
    resolve: {
        extensions: [".js", ".jsx", "*"],
        alias: {
            css: path.resolve(__dirname, "frontend/css"),
            js: path.resolve(__dirname, "frontend/js"),
            store: path.resolve(__dirname, "frontend/redux/store"),
            util: path.resolve(__dirname, "frontend/redux/util"),
            reducers: path.resolve(__dirname, "frontend/redux/reducers"),
            actions: path.resolve(__dirname, "frontend/redux/actions")
        }
    }
};

```

- set up your entry file (entry_file_name.jsx that correspondes to the entry file in webpack.config.js)
```
import React from "react";
import ReactDOM from "react-dom";

document.addEventListener("DOMContentLoaded", () => {
  const root = document.getElementById("root");
  ReactDOM.render(<h1>Welcome to BenchBnB</h1>, root);
});
```
 
## Rails Backend
- set up StaticPagesController with a root view containing 
```
rails g controller StaticPages
```
- inside /app/views/static_pages, create root.html.erb and make a root div
```
<div id="root"></div>
```

- update routes.rb file with root to static_pages#root
```
root to: 'static_pages#root'
```

- update StaticPagesController
```
class StaticPagesController < ApplicationController
  def root
    
  end
end
```

### Auth Backend
- create users database and user model 
```
rails g model user
```
- create users database and enter needed columns for user table and run migration
  - `rails db:create`
```
class CreateUsers < ActiveRecord::Migration[5.2]
  def change
    create_table :users do |t|
      t.string :username, null: false, unique: true
      t.string :session_token, null: false
      t.string :password_digest, null: false
      
      # .... additional columns.. 
      
      t.timestamps
    end
  end
end
```
  - `rails db:migrate`

- update user model
```
class User < ApplicationRecord
    validates :username, presence: true, uniqueness: true
    validates :password_digest, presence: true
    validates :session_token, presence: true, uniqueness: true
    validates :password, length: { minimum: 6, allow_nil: true }
    after_initialize :ensure_session_token

    attr_reader :password

    def self.find_by_credentials(username, password)
        user = User.find_by(username: username)
        return nil if user.nil?
        user.is_password?(password) ? user : nil
    end

    def password=(password)
        @password = password
        self.password_digest = BCrypt::Password.create(password)    
    end

    def is_password?(password)
        BCrypt::Password.new(self.password_digest).is_password?(password)
    end

    def reset_session_token!
        self.session_token = SecureRandom.urlsafe_base64(16)
        self.save!
        self.session_token
    end

    private
    def ensure_session_token
        self.session_token ||= SecureRandom.urlsafe_base64(16)
    end
end

```
- create users controller
```
rails g controller api/users
```
- update users controller
```
class Api::UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    if @user.save
      login(@user)
      render '/api/users/show'
    else
      render json: @user.errors.full_messages, status: 422
    end
  end

  private

  def user_params
    params.require(:user).permit(:username, :password)
  end
end
```
- update application controller

```
class ApplicationController < ActionController::Base
    helper_method :current_user, :logged_in?

    def logged_in?
        !!current_user
    end

    def login(user)
        @current_user = user
        session[:session_token] = user.reset_session_token!
    end

    def logout
        current_user.reset_session_token! 
        session[:session_token] = nil
    end

    def current_user
        @current_user ||= User.find_by(session_token: session[:session_token])
    end
end

```
- create sessions controller
```
rails g controller api/sessions
```
- update sessions controller
```
class Api::SessionsController < ApplicationController
    def create
        @user = User.find_by_credentials(
            params[:user][:username],
            params[:user][:password]
        )

        if @user
            login(@user)
            render '/api/users/show'
        else
            render json: ['Invalid username/password combination'], status: 401
        end
    end

    def destroy
        @user = current_user
        if @user
            logout
            render "api/users/show"
        else
            render json: ['Nobody signed in'], status: 404
        end
    end 
end
```

- update routes in routes.rb
```
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html

  namespace :api, defaults: { format: :json } do 
    resources :users, only: [:create]
    resource :session, only: [:create, :destroy]
  end

  root to: 'static_pages#root'
end
```

- before working on jbuilder files, edit config/environment.rb
```
Jbuilder.key_format camelize: :lower
```




# Deploying the app on Heroku

#### Setup instructions
1. Prepare your project for production
- Remove debuggers and console output
  -`cmd+shift+F` to search your project for debugger or console statements
- Move the `jquery-rails` gem to the top-level of your `Gemfile` so that it is available in both development and production
- Add engines and scripts to package.json
  - Make sure to use the versions installed on your computer. Use node -v & npm -v in termina
  ```
  // package.json
  {
    "engines": {
      "node": "12.5.0",
      "npm": "6.9.0"
    },
    "scripts": {
      "postinstall": "webpack"
    }
  }
  ```
- Execute bundle install and npm install. The postinstall script will take care of webpacking
  - Make sure everything installed correctly/nothing broke
- Commit changes to master. Remember, we should only push working repos to Heroku.
  - Remember to include node_modules, bundle.js*, and /public/assets in .gitignore
  ```
  git add Gemfile package.json other_files...
  git commit -m "Prepare for initial Heroku push"
  ```
2. Create a new Heroku app
- Create an account on heroku.com
- Create a new app
- Setup buildpacks
  - Settings tab in Heroku Dashboard
  - Add heroku/nodejs, then heroku/ruby
    - NB: Buildpacks run in the order specified, which is why it is important to add heroku/nodejs first. If heroku/ruby ran first, it would recompile our assets before creating a new bundle.js. We want bundle.js to be up-to-date before Rails recompiles our assets.
3. Install the Heroku CLI and make your initial push
- Install Heroku CLI
- Login with `heroku login`
- NOTE: if you are switching computers for development you will need to login on each switch, otherwise this should be a one-time action
- Add `heroku` remote to project repo
- Follow instructions in `Deploy` tab in Heroku Dashboard
  - You should already have a repo, so use `heroku git:remote -a appname` or `git remote add heroku https://git.heroku.com/appname.git`
- Push to Heroku remote: `git push heroku master`
- Check the status of your migrations with `heroku run rails db:migrate:status`
- Setup production database with: `heroku run bundle exec rails db:migrate`
- Optionally, include seed data with: `heroku run bundle exec rails db:seed`

4. Migrate static assets to the Asset Pipeline
- Put assets in `app/assets/`
- Images should be in `app/assets/images/`
- Use the Rails helpers (`image_url` for things in `app/assets/images/`, `asset_url` for anything in `app/assets/`) to generate the URL.
- For use in Rails views:
  ```
  <img src="<%= image_url('brent.png') %>" />
  ```
- For use in React:
  ```
  <!-- application.html.erb -->
  <script type="text/javascript">
    window.brentURL = "<%= image_url('brent.png') %>";
  </script>
  // your component
  render() {
    return <img src={window.brentURL} />;
  }
  ```
- For use in SASS
  ```
  image-url('brent.png')
  asset-url('brent.png')
  ```

### Common Gotchas
- Heroku build failed
  - This probably means there was a syntax error in one of your precompiled assets. Check the build output and try to find the culprit
- Heroku build was successful, but nothing shows up on the page and there are no console errors.
  - Failed to webpack your frontend. Double check your postinstall script, ensure there is only one scripts key in your package.json and make sure webpack is listed in your package.json dependencies
  - Try manually precompiling your assets by running `bundle exec rake assets:precompile RAILS_ENV=production`, then add, commit, and push back up to heroku
    - If this fixes your problem it means that you did not `.gitignore` your /public/assets folder. To permanently fix this problem and make sure Heroku handles this for you do the following
      - Add /public/assets to your `.gitignore` file
      - Run `git rm -r --cached` . from the root of your project
      - Add, commit, and push back up to Heroku
- "We're sorry, but something went wrong" error message appears on page load
  - Something is failing silently server-side. Use `heroku logs -t` to see your server logs and debug the issue
    - If you see an error related to PG (Postgres) you probably forgot to migrate. Run `heroku run bundle exec rails db:migrate`
  - If the heroku logs don't help, try running `heroku run rails c` - doing this might throw an error that will point you to the part of your code you need to fix
- Can't fetch any data
  - Make sure you created, migrated and seeded your database on Heroku
  - Run `heroku run rails db:migrate:status` to see which migrations you have and haven't run yet on Heroku
- Changing only capitalization when renaming a file.
  - Git will fail to recognize the change. Use `git mv -f <old_file_name> <new_file_name>` to force the update
- Asset pipeline not retrieving an asset, but is looking in the right directory (on `localhost`)
  - Assets are only precompiled when the server is starts up. Try restarting your server to initiate precompiling
- Missing package errors in console when viewing production site, but none on `localhost`
  - Likely using a global dependency in your project. This would be installed locally, but not on Heroku. Verify that all dependencies are included in your `package.json`
  
#### Helpful Console Commands
- `heroku logs -t` - opens running server log of your Heroku app
- `heroku run <command>` - can run any terminal command
  - `heroku run bundle exec rails console`
  - `heroku run bundle exec rails db:<cmd>`
- `heroku pg:psql` - connect to Postgres db (in lieu of `rails dbconsole`)
- `heroku pg:reset DATABASE_URL` - used to drop and reset your Heroku Postgres database (we don't have permissions to run `rails db:reset` and `rails db:drop` on Heroku)
- `DATABASE_URL` is a Heroku config variable we can reference from the command line
- Make sure to run `heroku run rails db:migrate` and `heroku run rails db:seed` to remigrate and reseed your database
- `heroku open` - opens your app in the browser
