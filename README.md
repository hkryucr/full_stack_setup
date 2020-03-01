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
```

- create a package.json file by running `npm init --yes` and create frontend folder including  actions, reducers, store, util, and components folders

- install frontend frameworks and libraries
```
npm install --save webpack webpack-cli react react-dom react-router-dom redux react-redux @babel/core @babel/preset-react @babel/preset-env babel-loader lodash redux-logger redux-thunk
```
- optionally, if you want to use css-loader, install as follows, which enables to use `import './another-stylesheet.css'` inside js/jsx files.
```
npm install --save-dev css-loader
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

- create 'webpack.config.js' file and set up the entrypoint, output path, and babel transpilation, set up the absolute paths, and include devtool: 'source-map'
```
const path = require("path");

module.exports = {
  context: __dirname,
  entry: "./frontend/your_entry_file_name.jsx",
  output: {
    path: path.resolve(__dirname, "app", "assets", "javascripts"),
    filename: "bundle.js"
  },
  module: {
    rules: [
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
      components: path.resolve(__dirname, "frontend/components"),
      actions: path.resolve(__dirname, "frontend/actions"),
      store: path.resolve(__dirname, "frontend/store"),
      util: path.resolve(__dirname, "frontend/util"),
      reducers: path.resolve(__dirname, "frontend/reducers")
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

