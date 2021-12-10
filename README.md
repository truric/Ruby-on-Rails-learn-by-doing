# Learning Ruby on Rails

This is my first hands on project, the goal is to learn by doing and to make personal notes as the project is being developed.

# INSTALATION

rails installer
ruby (override rails outdate version)
gem install rails version (update rails version)



# INTRO NOTES

Gemfile: dependencies
MVC architecture
Models: databases
Views: webpages
Controllers: logic
app/assets: css files, JV, images, etc
app/config/routes.rb



# CREATE NEW CONTROLLER

rails g (generate) controller home index
change route at: config/routes.rb, example: root 'home#index' page will load at / instead of default /home/index
check routes list: rails routes



# CREATING PAGES

when creating pages, create partials:
always name with underscore, example: _header.html.erb
call it as: <%= render 'home/header' %> rails will know the header is the _header.html.erb inside the views/home folder
<%= link_to 'Friend App', root_path, class:"navbar-brand" %>
link to makes a hyperlink and adds a placeholder 'text' with path

## create new page:

new file app/views/home/ example about.html.erb
add def about end on Controller
add get 'home/about' on config/routes.rb0
add hyperlink one page/navbar: <%= link_to 'About Us', home_about_path, class:"nav-link" %>
rails routes list, in order to see all routes, in order to use any just add _path at the end



# CRUD

rails g (generate) scaffold friends (model/table name) frist_name:string last_name:string email:string phone:string twitter:string (params and data type)
check db/migrate/* a new file is generated
supported data types:
:binary
:boolean
:date
:datetime
:decimal
:float
:integer
:primary_key
:string
:text
:time
:timestamp
https://guides.rubyonrails.org/v3.2/migrations.html

rails db:migrate: schema.rb is created
app/assets/stylesheets/scaffolds.scss can be deleted if it changed the webpages design
now you can access the CRUD already made at /friends
the routes were automaticly created as resouces :frends which contains all the friends possible routes
to add a link to Friends link, simply use friends_path, <%= link_to 'Friends', friends_path, class:"navbar-link" %>
to add a new Friend, use: new_friend_path



# DESIGN

using bootstrap, we can simply change the initial table tag> to (example) table class="table table-stripped"> or use another one from bootstrap documentation
same goes to thead> tag, using thead class="table-dark">
add/change buttons simply adding class: any-button-class after the path, example: <%= link_to 'New Friend', new_friend_path, class:"btn btn-secondary" %>



# NOTICE / ALERTS

there's a notice message when using crud, you can change the default message to a bootstrap designed alert
create a new partial at views/layouts, get an alert from bootstrap documentation
add <%= notice %> after the declared class
delete all the default notice on index/create/edit/new if there are any



# DEVISE GEM

other gems at rubygems.org
add devise gem to Gemfile
using device gem we can't simply run the command: bundle install
still on the devise gem page, on the Documentation: github.com/heartcombo/devise
check the topic Getting started
we already did the command bundle install
new we need to run, rails g devise:install
as prompted, we need to add that given line to the file at config/environments/development.rb
do the same to config/environments/production.rb if you have an heroku server or a server host
ensure you have set a root_url route, we got that set up already
ensure there are notice set on app/views/layouts/application.html.erb, got that covered
set up views: rails g devise:views
set a model for devise (user table): rails g devise user
rails db:migrate, devise_for :users is added at routes.rb, check rails routes to see the list

now we can set up hyperlinks on navbar (for example) using the path on the routes list, like:
Sign Up: <%= link_to 'Sign Up', new_user_registration_path, class:'nav-link' %>
Sign In: <%= link_to 'Sign Up', new_user_session_path, class:'nav-link' %>
Edit Profile: <%= link_to 'Sign Up', edit_user_registration_path, class:'nav-link' %>

in order to Sign Out, we need to do the same and also add method delete as follows:
<%= link_to 'Sign Out', destroy_user_session_path, method: :delete, class:'nav-link' %>

Now we can add logic, if user is signed in, do this, else that, example:
<% if user_signed_in? %>
grab both Sign Out and Edit Profile nav-items
<% else %>
grab both Sign Up and Sign In nav-items
<% end %>

Now it makes sense that, you should only be able to Add Friend if you are Signed In, let's do that:
grab the Add Friend code to the if user_signed_in? statement scope, and maybe the Friends list too, why not

bootstrapify all the possible forms adding form-group to form class and do the same to all possible fields adding class:"form-control", placeholder:"text"

at app/views/devise/shared/_links.html.erb we can bootstrapify the buttons adding "btn btn-secondary" on link_to code instead of changing buttons one page at a time

the Sign In page is located at app/views/devise/sessions/new.html.erb
the Edit Profile is located at .../devise/registrations/edit.html.erb
the Sign Up page is located at app/views/devise/registrations/new.html.erb


# RAILS ASSOCIATIONS
https://guides.rubyonrails.org/association_basics.html

possible types of associations:
belongs_to
has_one
has_many
has_many :through
has_one :through
has_and_belongs_to_many

app/models/friends.rb or user.rb (devise)
here we can set our association:

class Friend < ApplicationRecord
    belongs_to :user
end

Class User < ApplicationRecord
add: has_many :friends

add new param to Friends table so we can associate it to a user_id
run: rails g migration add_user_id_to_friends user_id:integer:index

now, if you check app/db/migrate/ there's a column and an index being created
now push it, rails db:migrate
if you check schema, there's a new param on friends table called user_id and its index

now that we added a new param, we need to add it to the form when creating a new Friend
devise has a few helpers, we can use the current_user to check for data, current_user.inspect you see almost as a json file, current_user.id is what we want, it returns only the integer needed
<%= form.number_field :user_id, id: :friend_user_id, class:"form-control", value:current_user.id, type:"hiddden" %>

the form is done, but if we try to create, an error will pop up, that's why we need to add the new param to the Controller
head up to app/controllers/friends_controller.rb
near the end at def friend_params, add :user_id to params.require(:friend).permit(:all_the_current_params, :user_id) and save it, done, now we can create new Friends

the way we've done this, each user has many friends, and each friends belongs to a user,
but if we log out, sign up a new user and log in, we see that this new user can see the previous user Friend list

back to the Friend controler, add:
before_action: :authenticate_user!, except: [:index, :show]
which means, if the user is NOT authenticated, it can only access the index and show defs from the controller
save it, test it on page with no logged in user, you can see that when trying to edit, destroy or even create, you get redirected to the login page

there's still a problem, i can edit Friends that don't belong to me (with different user_id) and when i do, the user_id changes to the current
------------
## NOT RECOMMENDED FIX:
a simple fix will be to hide the edit and destroyu from the index page, if the user logged in is not the same
<% if current_user.id == user_session %>
   edit
   destroy
<% else %>
   td>/td>
   td>/td> (just to avoid blank cells on table)
<% end %>

that is working on the index, but i can still get to the edit page of all created Friends
-----------
# RECOMMENDED FIX: add logic to Controller

bellow destroy function, add a new one:
def correct_user
   @friend = current_user.friends.find_by(id: params[:id])
   redirect_to friends_path, notice: "Not authorized to Edit this Friend" if @friend.nil?
end

on top, add a before action:
before_action :correct_user, only: [:edit, :update, :destroy]

on def new, comment the default code: @friend = Friend.new, and add:
@friend = current_user.friends.build

on def create, comment the default @friend = Friend.net(friend_params), and add:
@friend = curret_user.friends.build(friend_params)

now the routes are locked by user id and a notice will be prompted when reddirecting to friends_path

# OPTIONAL:
if we stil want to how the list of only the current user and not all the friends created even from other users
simply add logic to the index page:
<% if friend.user == current_user %>
   tr>
      td>all the params.../td>
   /tr>
<% end %>

obviously we can get to the show from other Friends by hand, so if needed, just change the Controller
----------



# DESIGN WEBPAGES LOGIC
https://github.com/heartcombo/devise#controller-filters-and-helpers
be free to mess around with the devise gem helpers:
user_signed_in?
current_user
user_session
maybe show the homepage as the list of Friends created, only if logged in
on routes, change the root to: root 'friends#index'
now on the friends home page, add logic before the list shows up:
<% if user_signed_in? %>
show all the Friends list params
<% else %>
show old home page or make another one
<% end %>

next, instead of "first name" and "last name", let's show only "name"
on friend index page, swap both for only th>Name/th>
below, on the table, get the <%= friend.first_name %> and <%= friend.last_name %> on one row only

next, if we want for the name to be clickable instead of having to click the "show" button, all we need to do is to add the show logic to the th>Name th> :
td> <%= link_to 'Show', friend %> <%= friend.first_name %> <%= friend.last_name %> /td>
instead of showing 'Show' text, we want the name, change it! :
td> <%= link_to friend.first_name + ' ' + friend.last_name, friend %> <%= friend.first_name %> <%= friend.last_name %> /td>

now we don't need the 'Show' button
on the friend index page, we can change the colspan to two: th colspan="2">/th> and delete the Show button td> below

we can get rid of the Edit as well, because we can access it onb the show page
so, colspan can be removed so just had a blank space to the th> tag, the edit button below can be deleted, the only action we have now is 'Destroy'

change the 'Destroy' text link to a button, maybe a bootstrap button, example, just add at the end of the td>: class:"btn btn-outline-danger btn-sm"

add a delete function/button when editing a Friend
copy the delete code from the index page
go to the edit page: paste the code and design the button to your needs, remember to add @ to friend


# HEROKU
rvm install 2.5.1 (because it's the version that was prompted)
rvm use 2.5.1
  change the ruby version on Gem file
  delete Gemfile.locl
bundle install
git add Gemfile.lock
git commit -am "Gemfile.lock ruby version"
git push
git push heroku main
----too many builds error----
heroku plugins:install heroku-builds
heroku builds:cancel
heroku restart
git push heroku main
----version ruby/rails error----
change version using rvm install/use x.x.x
----heroku wrong stack----
heroku stack:set heroku-18 (or -20 or the version prompted)