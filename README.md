# Omniauth::Census

## * This is a clone to hit the staging ENV for the Turing Census Application *

### * The Census OAuth provider only accepts callbacks using HTTPS, there will be config options at the bottom of this README to set up SSL Certs and a localhost to configure these settings*

Welcome to Census, and Omniauth strategy for Census. This gem makes it possible to log users into your application using their Turing Census loging credentials.

## Installation

Add this line to your application's Gemfile:

```ruby
# Gemfile

gem 'omniauth-census', git: "https://github.com/NZenitram/census_staging_oauth"
```

And then execute:

    $ bundle install

Or install it yourself as:

    $ gem install omniauth-census

## Usage
#### Step 1: Register your Application
Sign in to Census at [https://turing-census.herokuapp.com](https://turing-census.herokuapp.com).  
Visit [Your Census Applications](https://turing-census.herokuapp.com/oauth/applications) and register your app.
* Provide an application name
* Provide an application redirect uri (e.g. http://your-app.com/auth/census/callback)
* Optionally, provide a scope (note this feature is still in development)
* Add the ENV[`CENSUS_ID`], ENV[`CENSUS_SECRET`] you receive to your application's environment variables. For security, please ensure that these variables are not uploaded to GitHub or any other publicly available resource. If you need assistance with keeping these secret, consider using the [Figaro](https://github.com/laserlemon/figaro) gem. _(Figaro pronunciation: /fi.ɡa.ʁɔ/)_

#### Step 2: Configure OmniAuth
Create the following file:
`touch config/initializers/omniauth.rb`

Add the following configuration to the above file:
```ruby
# config/initializers/omniauth.rb

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :census, ENV["CENSUS_ID"], ENV["CENSUS_SECRET"], {
    :name => "census"
  }
end
```

#### Step 3: Setup Route
In your Rails application create the following routes:
```ruby
# config/routes.rb

get 'auth/:provider/callback', to: 'sessions#create'
```
#### Step 4: Create a Controller
In your Rails application create a controller that will handle the login process. For example:

`touch app/controllers/sessions_controller.rb`

In that controller include the following:

```ruby
# app/controllers/sessions_controller.rb

class SessionsController < ApplicationController
  def create
    census_user_info = env["omniauth.auth"]
  end
end
```

#### Step 5: Add Login Link

Add the following code to your desired view in order to create a Census Login Link

`<%= link_to 'Login with Census', '/auth/census' %>`

## Important note
Please note that in order to use the Census OmniAuth strategy, your application must be configured to handle secured HTTPS requests. This is not the default setting on typical Rails applications run locally. For instructions on configuring SSL on a development version of your application. The following steps will supply your application with and SSL cert and allow you to use HTTPS from your local host.

Add this line to your application's Gemfile:

```ruby
# Gemfile

gem 'thin'
```

The execute:

```
$ bundle install
```

Now work through the following steps:

```
## 1) Create your private key (any password will do, we remove it below)

$ cd ~/.ssh
$ openssl genrsa -des3 -out server.orig.key 2048

## 2) Remove the password

$ openssl rsa -in server.orig.key -out server.key


## 3) Generate the csr (Certificate signing request) (Details are important!)

$ openssl req -new -key server.key -out server.csr

## IMPORTANT
## MUST have localhost.ssl as the common name to keep browsers happy
## (has to do with non internal domain names ... which sadly can be
## avoided with a domain name with a "." in the middle of it somewhere)

Country Name (2 letter code) [AU]:

#### Just press enter to get past prompts until you reach:
...
Common Name: localhost.ssl
...
#### Fill out the Common Name field and skip the rest.

## 4) Generate self signed ssl certificate

$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

## 5) Finally Add localhost.ssl to your hosts file

$ echo "127.0.0.1 localhost.ssl" | sudo tee -a /private/etc/hosts


# 6) To start the SSL webserver open another terminal window and run

thin start -p 3001 --ssl --ssl-key-file .ssl/server.key --ssl-cert-file .ssl/server.crt
```

'Thin start -p 3001' will start your local host on port 3001. You will need to run the command in step 6.) in your application's directory. After it has started open your browser and visit 'localhost:3001'.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/bcgoss/omniauth-census.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
