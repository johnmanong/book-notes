# Ruby On Rails Tutorial
## *by Michael Hartl* ##

*This book provides a tutorial covering Ruby basics, Rails basics, deployment with Heroku, and building a twitter clone from the ground up. This book has multiple versions, these notes are for the Rails 4 version*



# working chapter #############################################################################



# Chapter 11 | title #############################################################################

#### Topics:
- following users
- relationship model
- web interface for following users
- status feed

#### References:
- [when to use rspec let](http://stackoverflow.com/questions/5359558/when-to-use-rspec-let)
- [Rails routing](http://guides.rubyonrails.org/routing.html)

- this chapter will add social layer to app
- users can follow and unfollow other users
- staus feed includes own and following users posts
- views to display followers and following
- model relationships betwen users
- introduce ajax
- some ruby/mysql trickery will be involved to make status feed

## ch 11.1 | The Relationship Model
- naive aproach: a user `has_many` followed users and `has_many` followers
- will be fixed by `has_many through`

### ch 11.1.1 | A problem with the data model (and a solution)
- there is a concept of the follower and the followed
- default Rails pluralization results in 'followeds' which is awkward
  - can call following, but that has oppoisite meaning
  - will use 'followed_users' and `user.followed_users` array
- simple `has_many` for followed users
  - each row of resulting followed users table would have all user info
  - this includes name, pw etc
  - very redundant
  - followers would similarly need another table
  - maintence is a nightmare, user info in up to 3 tables and multiple rows
- underlaying abstraction, in REST context: relationship
  - the relationship between users is what is created and destroyed
  - user `has_many :relationships` and hs many `followed_users` *through* relationships
  - can store ids only in relationships table, and get user info from user table
- getting users info
  - can get array of user ids from relationships table then lookup each user
  - Rails provides method: `has_many through`
  - e.g. `has_many :followed_users, through: :relationships, source: :followed`
- generate relationship model
  - `$ rails generate model Relationship follower_id:integer followed_id:integer`
  - remove factory if needed: `spec/factories/relationship.rb`
- update resulting migration to add indexes
  - follower_id
  - followed_id
  - follower_id, followed_id, enforce unique so users can't follow more than once
- run migration and update test db

### 11.1.2 | User/Relationship associations
- a user `has_many` relationships
- a relationship `belongs_to` both a follower and a followered user
- build a new relationship: `user.relationships.build(followed_id: ... )`
- start with basic validity test on relationship
  - create follower and followed uses
  - create relationship as mentioned above
  - check for should be valid
  - use let instead on instance var
- add test to user model to check for relationship
  - verify that user responds to relationship attr
- micropost table has a user id to identify users
  - this is known as a *foreign key*
  - foreign key for a User model object is `user_id`
- Rails infers foreign keys by default
  - converts class name to underscore
    - e.g. SomeClass -> some_class
  - checks for id named `<class>_id`
- in the Relationship/User association, we cannot rely on Rails default inference of the foreign key
  - in this case the foreign key is `follower_id` not `user_id`
  - in User model when declaring association, pass in param for foreign key
  - add `dependent: :destory` since relationships should be deleted when the user is deleted
- add tests for Relationship follower methods
  - check that relationship responds to follower and followed attrs
  - check that those attrs reference created objs
- update Relationship model
  - add belongs to association for follower and followed
  - fk is inferred from symbols
    - e.g. :follower -> follower_id
  - supply class name 'User' since there is not a follower or followed class
- all tests should be green

### 11.1.3 | Validations
- add tests for validation
  - if either follower or followed id is missing should not be valid
- add validation to Relationship model
  - `validats :follower_id, presence:true`

### 11.1.4 | Followed users
- start by implementing `followed_users`
- default implementation of `has_many through` would expect:
  - `has_many :followeds, through: :relationships`
  - this would get array using `followed_id` in the `relationships` table
  - this is awkward
- will use 'followed users' as plural of 'followed'
  - `user.followed_users`
  - overwrite default by using `:source` parameter
- update User model
  - `has_many :followed_users, through: :relationships, source: :followed`
- introduce helper methods in user model
  - `follow!(other_user)`
    - should always work (like `save!`)
    - will throw if failure
  - `following?(other_user)`
    - boolean
  - `unfollow!(other_user)`
    - unfollow a user
- update tests to anticipate methods
- update user model
  - self is optional

### 11.1.5 | Followers
- add `user.followers` method
  - all required info is in the Relationship table
  - same as followed_users, but the roles of follower_id and followed_id are switched
- add tests
  - add respond to tests for `:reverse_relationships` and `followers`
  - for followed user, assign other_user as subject and test @user is in followers
  - should be reversed roles as test for followed users
- will not create new db table for reverse relationship
  - simulate table by passing followed_id as the primary/foreign key
- add associations
  - when declaring the association, must assign 'Relationship' class explicitly
  - can leave off followers as source, since Rails will correctly assume foreign key follower_id
  - followers association should be similar but opposite to followed_users, but uses revser relationship
- all tests should pass


## 11.2 | A web interface for followers
- will implement basic interface for following/unfollowing
- seperate pages to should followers and following

### 11.2.1 | Sample following data
- will update `db:populate` rake task to add relationships
- refactor into helper methods to make users, posts and relationships
- make relationships method
  - get all users
  - get first user
  - get first user to follow users 3 - 51
  - get users 4 - 41 to follow user

### 11.2.2 | Stats and a follow form
- update profile and homepages to reflect followers and folled users
  - homepage and profile pages gets stats
  - profile pages get follow/unfollow buttons
- will use 'following' (same as twitter) although it can be ambiguous on its own
- update route file for `resources :users`
  - for each member (i.e. user), get following and followers
  - member means it responds to routes with user id in them
  - results in routes 'users/1/following' and 'users/1/followers
- another possibility, `collection`, does not require an id and acts on all users
  - e.g. `get :tigers` woudl respond to `users/tigers` and display all tigers in app
- note that this results in the use of 'following' and 'followed users' to identify the same group of users
  - this is less awkward from a routing, ui and resulting path var

| HTTP request | URL | Action | Named Route |
---------------------------------------------
| `GET` | /users/1/following | following | following_user_path(1) |
| `GET` | /users/1/followers | followers | followers_user_path(1) |

- add tests for stats partial
  - create another user
  - have that user follow first user
  - visit root
  - make sure that there are links with:
    - "0 following" and href: `following_users_path(user)`
    - "1 followers" and href: `followers_users_path(user)`
- create stats partial
  - if user instance var is nil, assign to current users (home and profile pages)
  - add followers and following count and labels
  - use association to get count
  - will have ajax implementation later
- update home view to render stats
- update custom.css.scss for stats styling
- create partial for follow/unfollow button
  - do not display if user is current users
  - if following, render unfollow
  - if not following, render follow
- update routes file for relationship routes
  - only `:create` and `:destory`
- create follow and unfollow partials
  - simple form to create/destory relationships, respectively
  - POST (create action) and DELETE (destory action)
  - follow form has hidden field to pass user id to create action
- update users/show view to include stats and follow form
- note, these do not work and tests to do pass
  - will implement standard method
  - will implement ajax method

### 11.2.3 | Following and followers pages
- wil create pages for displaying followers and users followed
- hybrid of user index page and user profile page
  - include user info in sidebar
  - include list of users (paginated)
- both follower and following pages require signin
  - update auth pages spec
- update user pages spec
  - make sure that for signed in user
    - title is correct
    - heading is correct
    - link is present
- two new actions in Users controller: `following` and `followers`
  - will render same partial (explictly call render)
  - sets title
  - finds user
  - finds either `@user.follow_users` or `@user.followers`
- create `show_users` partial
  - add title
  - display user info
  - if followers/following
    - add list of users and gravatar in grid
    - render list of users by calling `render @users`
- less specs fail but still 3 failures

### 11.2.4 | A working follow button the standard way




#############################################################################



# General Notes ###################################################################################

## HEROKU ###################################################################################

    $ heroku create                  // create heroku side repo
    $ git push heroku master         // push to heroku
    $ heroku rename railstutorial    // rename repo on heroku (optional, must be unique)
    $ heroku logs                    // view logs

## Postgres ###################################################################################

    $ \l                              // show all databases
    $ \c <database name>              // connect to databse
    $ \d                              // show all in db
    $ \dt                             // show all tables in db
    $ SELECT * FROM <table name>;     // show contents of table
    $ \q                              // quit


# Chapter 1 | From Zero to Deploy #############################################################################
- creating git repo and pusing
$ git remote add origin https://github.com/<username>/<appname>.git
$ git push -u origin master

- creating and pushing to heroku
* in production.rb
	...
	config.serve_static_assets = true
	...

$ heroku create						// create heroku side repo
$ git push heroku master			// push to heroku
$ heroku rename railstutorial		// rename repo on heroku (optional, must be unique)


# Chapter 2 | A Demo App #############################################################################

- creating a new app
$ rails new <appname>

- to bundle without installing production gems
$ bundle install --without production

- scaffolding & db migration(ex)
$ rails generate scaffold User name:string email:string
$ bundle exec rake db:migrate

- in model, use 'validates' to add constraints
  - ex: 'validates :content, length { maximum: 140 }'


- associations
  - one to many (ex: User has_many :microposts)
  - owned by (ed: Micropost belongs_to :user)
  - allows calls such as 'User.microposts' to get all posts for a given user



# Chapter 3 | Mostly static web pages #############################################################################

- start of sample app (rest of tutorial)
- design process (Behavior Driven Design, BDD)
  1) write test
  2) fail test
  3) write code
  4) pass test

- layout
  - application.html.erb in layout directory used to build every page
  - will wait for each view at the <% yield %>
  - can provide view specific variables to the layout using the yield+var in layout, and provide helper in view
    - in layout: '<%= yield(:title) %>'
    - in view: '<% provide(:title, "some title") %>'

- rspec testing
  - rspec uses Domain Specific Language (DSL)
  - Capybara added to spec_helper.rb
    - (in rspec.config do): config.include Capybara:DSL
    - allows use of functions like 'visit' and variables like 'page'
  - in this chapter, used testing to verify content and title when visiting pages
  - to generate test for controller
    $rails generate integration_test <controller name>
  - use let helper to define variables
    - declaration: 'let(:title_base){ "Ruby on Rails Tutorial Sample  App | " }'
    - in test: "#{:title_base}Home"

- postgres local installation 
  - ref:  https://devcenter.heroku.com/articles/heroku-postgresql#local-setup
          http://www.glom.org/wiki/index.php?title=Initial_Postgres_Configuration
          http://railscasts.com/episodes/342-migrating-to-postgresql
          TAPS - to move db from one to another

  - $ sudo -s -u postgres
    - $ createuser -P
      - enter name
      - enter pw
      - super user = 'y' if intial user
  - $ createdb <name>
  - $ psql -l (for list of dbs)
  - $ psql (db name)
   - '\q' to quit


  psql: jong/mango

- advanced setup
  - pre 1.11 versions of RVM will require additional work to remove the need for 'exec bundler' before 'rspec spec'
  - guard gem allows you to watch for file changes and run corresponding tests (watching views, controllers, and sessions_controller)
  - spork runs a local server for testing to eliminate overhead in loading rails instance when running spec
    - will need to bounce this server when bouncing rails
  - using guard and spork together, you can watch files, watch for changes and run spec test on a guard initiated spork server to fast continuous testing...NEAT!
  - can also run tests inside Sublime Text
    - need sublime-text-2-ruby-tests package
    - git clone https://github.com/maltize/sublime-text-2-ruby-tests.git RubyTest

  - typical test/coding process
    1) Start Spork in a terminal window.
    2) Write a single test or small group of tests.
    3) Run Command-Shift-R to verify that the test or test group is red.
    4) Write the corresponding application code.
    5) Run Command-Shift-E to run the same test/group again, verifying that it’s green.
    6) Repeat steps 2–5 as necessary.
    7) When reaching a natural stopping point (such as before a commit), run rspec spec/ at the command line to confirm that the entire test suite is still green.


# Chapter 4 | Rails Flavored Ruby #############################################################################

- Rails console (rails c)

- Strings
  - single or double quotes
  - string interpolation MUST use double quotes
    e.g. "This string has a #{variable}"
  - single quotes escape chars within it
  - string.upcase : capitalizes each letter in string
  - string.downcase : downcases each letter in string

- printing to screen
  - 'puts' appends a new line
  - 'print' does not

- Objects and Message Passing
  - everything is an object
  - messages are passed to objects
    e.g. "string".length  -> 6
  - messages ending with '?' return boolean values
    e.g. "".empty? => true
  - nil is also an object, which can have messages passed to it
    e.g. nil.to_s => ""
  - cannot pass any message
    e.g. nil.empty? => blows up!
  - can check for nil-ness
    e.g. nil.nil? => true
    e.g. "this".nil? => false
  * can also use .blank? which checks for empty string or nil
    e.g. "".nil? => false, "".blank? => true
    e.g. nil.nil? => true, nil.blank? => true

- can use conditional statements in normal stucture (if else end) and compact conditional after statment
  e.g. puts "hello if true" if "this".empty?
  - 'unless(...)' == 'if(!...)'

- ruby functions have implicit returns; that is, they will return the last statement evaluated

- Arrays
  - use the '.split' message to create an array from a string
    e.g. "this is a string".split(" ") => ["this", "is", "a", "string"]
  - accessing arrays
    - typcial zero offset indices  
      e.g. a[0] => first element in array
    - negative indices wrap
      e.g.  a[-1] => last element in array
    - english synonyms
      e.g. a.first == a[0], a.last == a[-1]
  - arrays also have a '.length' message 
    e.g. a.length => 3
  - manipulating the array
    - a.sort : simple sort
    - a.reverse : simple reverse sort
    - a.shuffle : randomize order of elements
    - use 'a.(message)!' to persist changes on array
    - NOTE: cannnot sort arrays which have more than one type (e.g. strings and numbers)
  - adding to an array
    - a.push(6) : adds 6 to end of array
    - a << 6 : adds 6 to the end of the array
    - a << 6 << 7 : add 6 to the array, then 7
  - consolidating arrays
    - a.join : creates string of all elements stringified and concatonated
    - a.join(', ') : same as above but delimits with a ', '
  - creating an array of strings
    a = %w[this is the string array]
    => ["this", "is", "the", "string", "array"]

- Ranges
  - declared using boundries of range
    e.g. (1..9) => range form 1 to 9 (inclusive)
  - (1..9).to_a : creates range from 1 to 9 and converts to an array
  - a[1..4] : access elements 1 thru 4 of the array
  - a[1..-1] : access elements from 1 to end
  - can also be created for letters
    e.g. ('a'..'c').to_a => ['a', 'b', 'c']

- Blocks
  - blocks are closures (anonymous functions) with data passed to them
  - '**' is the notation for power
    e.g. 1**2 == 1^2
  - 'map' can be applied to an array or range and returns the result of applying the block to each element
    e.g. %w[a b c].map{ |char| char.upcase } => ["A", "B", "C"]

- Example from Chapter 1:
  ('a'..'z').to_a.shuffle[0..7].join

  1) create a range from 'a' to 'z'
  2) create array from range
  3) shuffle order of elements in resulting array
  4) take first 8 elements from now shuffled array
  5) join those 8 elements (letters) into a single string

### 4.3.3 | Hashes and Symbols
  - like arrays, but indexed by more than just ints
  - does not guarantee preservation of order (if needed use array)
  - use symbols for keys, less baggage than strings
  - 'puts :name.inspect' == 'p :name'

### 4.3.4 | CSS Revisited
  - ruby method calls, parens are optional
  - when hash is last arg, curly braces are optional
  - cannot use '-' in symbol names


## 4.4 | Ruby Classes
  - classes are instantieated to create objects (everything is an object!)

### 4.4.1 | Constructors
  - i.e. "foo = String.new('bar')"
  - similarily, "Array.new, Hash.new"
    - "Hash.new(0)" sets a value for unknown keys
  - "class method" : when a method is called on the class itself
  - "instance of a class" : object return when calling the class constructor
  - "instance methods" : methods called on the instance (object) of the class

### 4.4.2 | Class Inheritance
  - call "obj.class" to get class back; "obj.class.superclass" returns the super class (parent class)
    - i.e.
      s = String.new
      s.class 
        > String
      s.class.superclass
        > Object
  - to define a class which inherits from another: "def Word < String ..."

### 4.4.3 | Modifying a Built in Class
  - Ruby allows you to open and extend built in classes by simply redefining them
    - i.e.
      class String 
        def palindrome?
          self == self.reverse
        end
      end
  - while this is very powerful, be careful, you should have a REALLY GOOD reason to do so
  - Rails does this out of the box (i.e. present?, blank? are not std Ruby)

### 4.4.4 | A Controller Class
  - controllers are classes too
    - i.e. 
      StaticPagesController < ApplicationController < ActionController::Base < ActionController::Metal < AbstractController::Base < Object < BasicObject
  - controllers are classes too, but actions aren't intended to return anything
  - Rails classes are not always pure Ruby classes in this sense

### 4.4.5 | A User Class
  - "attr_accessor" : attribute accessor, creates setters and getters for instance variables (@var)
  - "@var" : instance variable
    - in Ruby, generally used for vars which need to be available throughout a class
    - in Rails, primarily used for persisting vars to the views
    - always begin with '@'
    - returns 'nil' when not defined
  - "initialize" : default Ruby constructor
    - default args is empty hash
    - can assign values through arg hash
  - include class file in rails console: "require './example_class_at_root'"

## 4.5 | Conclusion
  - will use classes in ch5 and beyond!

## 4.6 | Exercises
  1) create a shuffle string method
    def string_shuffle(s)
      s.split('').shuffle.join
    end

  2) extend the string class to include the shuffle method
    class String
      def shuffle
        self.split('').shuffle.join
      end
    end

  3) 
    Create three hashes called person1, person2, and person3, with first and last names under the keys :first and :last. Then create a params hash so that params[:father] is person1, params[:mother] is person2, and params[:child] is person3. Verify that, for example, params[:father][:first] has the right value.

    irb(main):015:0> person1 = {first: "joe", last: 'smith'}
      => {:first=>"joe", :last=>"smith"}
    irb(main):016:0> person2 = {first: 'jane', last: 'smith'}
      => {:first=>"jane", :last=>"smith"}
    irb(main):017:0> person3 = {fist: 'child', last: 'smith'}
      => {:fist=>"child", :last=>"smith"}
    irb(main):018:0> params = {father: person1, mother: person2, child: person3}
      => {:father=>{:first=>"joe", :last=>"smith"}, :mother=>{:first=>"jane", :last=>"smith"}, :child=>{:fist=>"child", :last=>"smith"}}
    irb(main):019:0> params[:father][:first]
      => "joe"

  4) Read about Ruby has merge
    http://www.ruby-doc.org/core-2.0.0/Hash.html

    - merge : (new hash)
      - returns new hash which is the merge two hashes, with the hash passed as arg used for key collisions
        a = {a: 1, b: 2}; b = {b: 3, c: 4}; a.merge(b) #=> {a: 1, b: 3, c: 4}
      - can also pass block and manipulate collisions explicitly
        merge(other_hash){ |key, oldVal, newval| <do stuff> }

    - merge!
      - adds contents of other hash to has merge! is called on
      - same signiture as above

  5) Follow Ruby Koans
    http://rubykoans.com/

      - rewrite hash it is called on


# Chapter 5 | Filling in the Layout #############################################################################

- Topics:
  - styling
    - bootstrap
    - sass
  - partials
  - asset pipeline
  - routes
  - revisit Rspec
  - user sign up

- References:
  - Mockingbird (https://gomockingbird.com/)
  - Google HTML5 Shim (http://html5shim.googlecode.com/svn/trunk/html5.js)
  - Twitter bootstrap ()

## 5.1 | Adding Some Structure
  - "wireframes" : static mocks of page layout and design
  - Mockingbird is an online wireframing tool
  - creating new branch 'filling-in-layout'

### 5.1.1 | Site Navigation
  - IE conditional comments
    - i.e. <!-- [if lt IE 9] ... <!endif-->
  - html5 shim from Google of < IE 9
  - new tags for html5, more specific divs -> 'header', 'nav', 'sections'
  - adding bootstrap specific html classes for future styling
  - as mentioned in ch3, the 'yeild' method inserts the contents from the page into the layout
  - 'image_tag' : generates image tag, defaults 'alt' to filename (sans extension)

### 5.1.2 | Bootstrap and Custom CSS
  - Twitter bootstrap build using LESS CSS (similar to SASS)
  - 'bootstrap-sass' gem translates bootstrap to Sass
  - ** installed 'SCSS' and 'SASS' packages into sublime
  - need to update 'config/application.rb' for asset pipeline to work with bootstrap
    - (inside of applicaiton class) -> 'config.assets.precompile += %w(*.png *.jpg *.jpeg *.gif)'
  - add css stylings onto of bootstrap defaults

### 5.1.3 | Partials
  - refactor page structure into logical chunks, uncluttering layouts/views
  - user the 'render' method to render partials
  - by convention file names start with an '_'

## 5.2 | Sass and the Asset Pipeline
 - Asset Pipeline helps manage javascript, css, and image files
 - Sass is available by default in the asset pipeline

### 5.2.1 | The Asset Pipeline
  - "asset directories" : default directories where js, css and image assets should go
  - "manifest files" : 
  - "preprocessor engine" : 

  - Asset Directories
    - the three conanical directories: 
      - "app/assets" : assets specific to present applicaiton
      - "lib/assets" : assets from libraries, written both dev team
      - "vender/assets" : assets from third party vendors
    - each 'assets' directory has three sub-directories
      - "javascripts"
      - "stylesheets"
      - "images"

  - Manifest Files
    - tells rails (via the sprokets gem) how to combine assets into single files (not for images though)
    - can be found in "app/assets/stylesheets/applicaiton.css"
    - 'require' are found in the comment block at the top fo the file
    - default requires
      - "*= require_tree ." : includes all files in app/assets/stylesheets
      - "*= require_self" : incless 'application.css' itself

  - Preprocessor Engines
    - after assets are assembled, files are run through preprocessor engines so manifest file can combine and deliver to browser
    - commone preprocessor engines: '.scss' (Sass), '.coffee' (CoffeeScript), '.erb' (embedded Ruby)
    - these can be chained
      - starts with last extension
      - e.g. 'foobar.js.erb.coffee' will run though the CoffeeScript preprocessor first, then ERb

  - Efficiency in Production
    - compilation into single files also includes minification, which makes the file smaller but difficult to read
    - does not work well for caching, if each page has its own file
    - precompiling assets is best for production, but can be turned off in development for developers sake (debug_assets=true ?)

### 5.2.2 | Syntatcially Awesome Stylesheets
  - improves css, 'nesting' and 'variables' ('mixins' covered in a later chapter)
  - .scss is strictly a superset of CSS (only adds)
  - every valid css file is a valid scss file

  - Nesting
    - common to nest selectors which share a common root selector
    - use '&' when explicitly referring to nesting selector, otherwise resolved to 'nesting nested'
    - i.e.
      div { ... }        ->       div {
      div a { ... }      ->         a { ... }
      div:hover { ... }  ->         &:hover { ... } }

  - Variables
    - prefixed with '$'
    - assigned like any other css property
      - e.g. '$lightGray: #999; div { color: $lightGray; }
    - LESS (used by Bootstrap) uses '@' to denote variables
    - 'sass-bootstrap' gem allows use of Bootstrap variables defined in LESS to be referenced in SASS syntax
      - ** variables only available in files which bootstrap is imported


## 5.3 | Layout Links
  - rails assosciates a semantic name for the url and path associated with route
  - variables (foo_path, foo_url) can be used instead of hardcoded string ('/foo')

### 5.3.1 | Route tests
  - use this to refactor spec test
  - e.g. '/static_pages/about' -> about_path
  - specs fail bc routes are not in correct form to generate this var

### 5.3.2 | Rails Routes
  - current "get 'static_pages/help'" does not generate route vars
  - change to "match '/help', to: 'static_pages#help', via: 'get'"
    - tells rails which path to match, which controller to route to, and http method
    - generates named route 'help_path'
      - help_path   -> '/help'
      - help_url    -> 'http://localhost:3000/help'
  - can define root path same way or special syntax
    - "root 'static_pages#home'"
      - root_path   -> '/'
      - root_url    -> 'http://localhost:3000/'

### 5.3.3 | Named Routes
  - update stubbed href from "'#'" to appropriate "foo_path"
  - simplifies layouts

### 5.3.4 | Pretty Rspec
  - use named route var in tests for cleanup
    - use before {} at the top of describe block to DRY up tests
      - this block runs before each test case in block and is used for shared initialization
  - test descriptions and requirements are redundant
    - subject {} can also DRY up tests
    - use subject {} to associate a variable with 'should'
      - e.g.
        OLD:
        it "should have the content 'Sample App'" do    # description
          expect(page).to have_content('Sample App')    # requirement
        end

        NEW:
        subject { page }
        it { should have_content('Sample App') }
  - when using blocks like 'subject' and 'before' declare as high as possible for max DRY
  - create file for spec utility functions
    - essentially duplicte from ch4 (applicaiton_helper.rb)
    - create utility to return full title
    - two implementations right now

## 5.4 | User Signup: A first step
  - will create second controller, Users controller
  - will complete work to allow users to sign up in ch 6 and 7

### 5.4.1 | Users Controller
  - create new controller
    - generate:
      $ rails g controller Users new --no-test-framework
    - resulting controller: app/controllers/users_controller.rb
    - resulting view: app/views/users/new.html.erb

### 5.4.2 | Signup URL
  - generated route path '/users/new', but want '/signup'
  - start with integration testing
    - generate:
      $ rails g integration_test user_pages
  - can run tests several ways
    $ bundle exec rspec spec/requests/user_pages_spec.rb    # run spec file
    $ bundle exec rspec spec/requests/                      # run all specs in requests dir
    $ bundle exec rspec spec/                               # run all specs
    $ bundle exec rake spec                                 # run all specs (needs test db, TODO ch 7)
    $ bundle exec rake                                      # run all specs (needs test db, TODO ch 7)
  - add route for "match 'signup"
  - also leave "get 'users/new'"
    - auto generated
    - does NOT follow REST
    - needed to get 'users/new' routing to work
    - will get cleaned up in ch7
  - changed title and content in new.html.erb to pass spec
  - added 'signup_path' to link on homepage (replace stub)

## 5.5 | Conclusion
  - this chapter, adding and cleaning up layouts and routes
  - next: sign up, sign in/out
  - then: add microposts
  - then: follow other users
  - update git repo
    - merge feature branch

  - TODO:
    * setup Heroku
    * push to Heroku

## 5.6 | Exercises
  1) Use Capybara's 'have_selector' method to check for html elements, and additionally check the text of those elements. Used to replace 'have_content' which can be too broad

  2) DRY up spec test for static pages using Rspec's shared examples functionality and let. 'let' allows you to define local variables (as opposed to instance variables which are created upon assignment). 'shared examples' allow you to share a set of common tests which can use local vars

  3) Add spec test to click through links on page and check title

  4) Create 'application_helper_spec.rb' to test helper function for full title. remove full_title implementation from utilities.rb and include application helper instead


# Chapter 6 | Modeling Users #############################################################################

#### Topics
  - user model
  - user validation
  - secure pw

#### References
  - Rubular
    - Ruby regex scratchpad
    - http://www.rubular.com/



- chapters 6-9, will create full login/authentication system
  - ch 6, create data model
  - ch 7, sign up and create profile page
  - ch 8, sign in and out
  - ch 9, protect pages from improper access
- pre-rolled auth solutions
  - Clearance
  - Authlogic
  - Devise
  - CanCan
  - OpenId*
  - OAuth*
    * general solutions are not Rails specific
- create new branch 'modeling-users'

## 6.1 | User Model
- end goal to create signup, currently have nowhere to store it
- Rails default data struct for data model is model
  - M in MVC
- persistence through database
- default lib to interact with the database is ActiveRecord
  - comes from 'active record pattern' named in Patterns of Enterprise Application Architecture by Martin Fowler.
  - allows interaction with data w/o having to use underlying SQL lang used by relational dbs
- Rails features 'migrations' which allows data definitions in Ruby, no SQL Data Definition Language (DDL) needed
- Allows developer to abstract underlying db technology
  - current setup:
    - SQLite for development
    - PostgresSQL for deployment on heroku

### 6.1.1 | Database Migrations
- previously defined a User class with name and email fields, but lacked persistence
- User model will have name and email (acts as unique username)
  - implemented via Ruby's 'attr_accessor' method
- Rails models do not have to have attrs explicitly defined
  - uses relations db with tabels, data rows and attr columns
  - will defined a user table with name and email columns
  - ActiveRecord will figure out what attrs the User obj will have from this table
- used to generate generate User controller: 
  `$ rails generate controller Users new --no-test-framework`
- analogous method to generate model:
  `$ rails generate model User name:string email:string`
  - model names are singular, as opposed to controller names which are plural
  - defines model with two string attrs, name and email
  - creates migration to update db
  - creates users table with name and email columns
- migrations are prefixed with a timestamp to avoid collisions
- migration has 'change' method, which uses 'create_table'
- 'create_table' takes a block and one block var (defaults to 't')
- uses plural 'users', which follows Rails convention: Model is singular, db is plural
- 't.timestamps' creates two magic columns, one for 'created_at', one for 'updated_at'
- once migration is defined/generated, need to run migration to affect db
  `$ bundle exec rake db:migrate`
- migrations can also provide a way to roll back or undo migration
  - to undo one migration: `$ bundle exec rake db:rollback`
  - uses 'drop_table' method
- 'change' method can infer that 'create_table' and 'drop_table' are opposites
- for irreversable migrations:
  - seperate 'up' and 'down' methods must be defined
  - e.g. deleting a column in a table

### 6.1.2 | The Model File
- migration updated 'db/development.sqlite3' file with 'users' table, which has columns:
  - id
  - name
  - email
  - created_at
  - updated_at
- also creates the 'User' model in 'app/models/user.rb'
  - this model inherits from 'ActiveRecord::Base' class

### 6.1.3 | Creating User Objects
- run Rails console in sandbox (no changes to db): `$ rails console --sandbox`
- Rails console automatically loads Rails env, so models are readily available
- create new User object: `>> User.new`
- with no args, all attributes are 'nil'
- ActiveRecord allows objects to be created with an initialization hash
  `>> User.new(name: 'John', email:'john@example.com')`
- creating object via '.new' does NOT touch the database
  - to save, call '.save' on the object
  - returns 'true' when success, 'false' when failure
- 'id', 'created_at', 'updated_at' were all nil when the obj was created in memory
  - when obj is saved to db, these columns are auto-populated and model attrs set
- notes that timestamps are in UTC time
- as with User class, attrs of the instances of the User model can be accessed via dot notation
- can create obj and save to db in one step:
  `user.create()`
  - this return the obj itself instead of true/false
  - (?) what is returned if it cannot be saved to db?
- opposite of create is destroy
  `user.destroy`
  - destroy method returns obj
  - destoryed objs are still available in memory, but not in db
  `x.destory
   => #<User ... >
   x
   => #<User ... >
   User.all
   => <does not include x>`
- how do we ensure objs are destoryed or that non-destoryed objs are in db?

### 6.1.4 | Finding User Objects
- can search table for rows with matching id
  `User.find(<id>)`
  e.g. `User.find(1)
        => #< User id: 1 ... ?`
- when no record was found, an exception is raised (specific exception: ActiveRecord::RecordNotFound)
- can also find by specific attrs:
  `User.find_by_<attr>(<attr value>)`
  e.g. `User.find_by_email('john@example.com')`
- starting in Rails 4, the preferred way to do this is by using 'find_by' and specifying field in arg
  `User.find_by(email: "john@example.com")`
- 'find_by' is inefficient over large sets of data (more on this later)
  - database indices! (6.2.5)
- other find methods
  - 'User.first' : returns first record in table
  - 'User.last' : returns last record in table
  - 'User.all' : returns all records in table in an array

### 6.1.5 | Updating User Objects
- two basic ways to update objs which

1) direct assignment + save

    >> user                                 # this is a ref to entry in db
    => #< User ... >
    >> user.email = "john@somewhere.com"
    => "john@somewhere.com"
    >> user.save
    => true

  - will not persist if you forget to call '.save'
  - updates 'updated_at' column automatically

2) '.update_attributes'

    >> user.update_attributes(name: 'dude', email: 'dude@abides.net')
    => #< User ... >
    >> user.name
    => "dude"
    >> user.email
    => "dude@abides.net"

  - updates and saves in one command
  - returns true if success
  - if validation fails (such as req'd pw), 'update_attributes' will fail
  - use singular 'update_attribute' to update one attribute

- 'user.reload' can be used to reload obj in memory from db (also 'user.reload.email')


## 6.2 | User Validations
- currently, there are no validation checks on User model attrs
  - name should be non-blanks
  - email should be in a particular format, and unique (since email is used as user name)
- ActiveRecord allows for checks through validations
- common validations
  - presence
  - length
  - format
  - uniqueness
  - confirmation
- violations give error messages when validations fail

### 6.2.1 | Initial User Tests
- user spec (app/spec/models/user_spec.rb) is practically blank since we passed the `--no-test-framework` flag at generation
- 'pending' is a placeholder to inidicate we should fill it in
- we can run the test by running the following:


    $ bundle exec rake db:migrate                   # run migrations
    $ bundle exec rake test:prepare                 # setup test db
    $ bundle exec rspec spec/models/user_spec.rb    # run spec

- if tests are failing, try reseting test db using `rake test:prepare`
- fill in spec test for model
  - use 'before' method to create instance var of example User
  - user 'subject' method to set created instance var as default subject
  - test if subject responds to ':name' and ':email'
- as is, tests are not entirely useful, since before block would throw exception if one attr didn't exist
- brings attention to reader that these exist
- 'respond_to' method uses Ruby's 'respond_to?' method
  - accepts a symbol
  - returns *true* if obj responds to method (e.g. user.respond_to(:name))
  - return *false* if the obj does not respond to method (e.g. user.respond_to(:foo))
- *Recall* Ruby uses the '?' to annote boolean methods
- two styles of writing tests

1) more verbose

    it "should respond to 'name'" do
      expect(@user).to respond_to(:name)
    end
2) more succinct

    it { should respond_to(:name) }

### 6.2.2 | Validating Precense
- most elementary of validations, tests for existence of attr
- start with validation of existence of **name** attr
- to do this, use **validates** method with arg **presence: true**
  - this is a one element options hash
  - *recall* {} is optional for hash passed as final arg
- empty string ('') or whitespace (' ') will return false for **presence** test
- if validation fails, **.save** will return false
- to test validation: `user.valid?`
- an errors obj is generated on failure
  - generated by either **.save** or **.valid?**
  - to access: `user.errors.full_messages`
- remember, when following TDD, write your test to fail first
  - comment out validation so we can do this
- spec test can check for validity with **be_valid**
  - here we will test for valid modela and invalid (name missing)
- for a valid User obj, name and email should be present
  - write spec test for invalid object when email is ' '
  - write validation for User class to check for presence of email attribute

### 6.2.3 | Length Validation
- add max length to name, 50 chosen 'out of thin air'
- add failing spec test (use string multiplication for convenience)
- add validation, spec test should now pass

### 6.2.4 | Format Validation
- email format, require that email fits a certain format: user@example.com
- tests and validation will **NOT** be exhaustive, just good enough to accept/reject basics
- use **%w[ ]** method to create arrays from strings
- test for basic valid email:
  - uppercase
  - underscores
  - compound domains
  - first.last (corp)
  - two letter top domain (jp)
- use *regex* to validate email pattern
- format validation: `format: { with: <regex> }`
  - only strings which match the pattern are valid
- regex
  - terse language for matching text patterns
  - Rubular is a great tool (http://www.rubular.com/)
- email regex:

| REGEX                                  | description
| -------------------------------------- |-------------
| `/\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i` | full valid email regex
| `/`                                    | regex start
| `\A`                                   | string start
| `[\w+\-.]+`                            | at least 1 char, plus, hyphen or dash
| `@`                                    | literal 'at sign'
| `[a-z\d\-.]+`                          | at least 1 letter, digit, hyphen or dot
| `\.`                                   | literal dot
| `[a-z]+`                               | at least 1 letter (lower case*)
| `\z`                                   | string end
| `/`                                    | regex end
| `i`                                    | case insensitive*

- current regex allows 'user@example..com', will be fixed in exercise
- official email regex exists, but let's some strange stuff through, such as, "john ong"@example.com

6.2.5 | Uniqueness Validation
- **:unique** option on the **validates** method
- **MAJOR CAVEAT** : you've been warned
- start with tests
  - add duplicate user
  - will have to save to db to validate uniqueness
  - user **user.dup** to create duplicate
- update validates method to include check for uniqueness
  - checks for duplicate emails should be case insensitive
  - currently is not the case
- convert email to uppercase using **.upcase** to check
- **:uniqueness** option also takes a **case_sensitive** arg
  - defaults to true
- CAVEAT: using **validates :uniqueness** does not always work!
  - if a user was to double submit request, both request can pass validation and save
  - need to enforce at db level
  - create indices for rows and enforce uniqueness on those
- generate migration to update current users table:

    $ rails generate migration add_index_to_users_email

- this migration is not defined so we will need to fill in
  - users **add_index** method
  - add index to **email** column of **users** table
  - index itself does not enfore uniqueness, but can set flag
- could have also updated existing migration
  - would have to roll back migration
  - Rails way is to have new migration for each change
- after migration is defined, run it: `bundle exec rake db:migrate`
- if failing, exit 'rails console --sandbox'
- inspect 'db/schema.rb' to see changes
- what about case?
  - need to convert to downcase to ensure consistent casing between entries
  - not all db adaptor indices are case insensitive (sqlite3 and postgres differ, for example)
  - will use *callback* **before_save** to downcase email prior to saving
  - will later use *method reference* convention
- removing email validation, we can verify this
  - postgres thows a duplicate index error
  - adding back validation, '.save' will fail
- unique indices fix **find_by** issue
- database indices
  - when creating table, do we need to each
  - in our data model, need to compare email to email in all rows of database
  - this is called *full table scan* and does not scale well
  - adding a database index fixes this, since an index can be inferred from the email; once you have the index lookup is fast
  - think book index: instead of scanning text for match, look it up in the reference to find pages

## 6.3 | Adding a secure password
- will implement pw with confirmation and store encrypted version in db
- add a way to authenticate a user based on pw
  - encrypt submitted pw
  - compare encrypted input to db
  - even if db is compromised, pws are encrypted
- most work done by Rails method **has_secure_password**

### 6.3.1 | An encrypted password
- add **password_digest** column to user table
  - digest comes from terminology of *cryptic hash functions*
  - storing encrypted pw means someone can't sign in even if they get a copy of the database
- make use of state-of-the-art hash function called *bcrypt*
  - irreversibly encrypt to form pw hash
  - install gem `'bcrypt-ruby', '3.1.2'`

  TODO: install gem (check version)
  

- add spec test to check for this column (test should fail)
- add migration `$ rails generate migration add_password_digest_to_users password_digest:string`
  - ending the naeme with '_to_users' + column allows Rails to auto-gen entire migration
- run migration, prepare test db, and all specs should be green

### 6.3.2 | Password and confirmation
- conventional to let the model handle pw, user ActiveRecord to enfore constraint
- **has_secure_password** adds two attributes to the model
  - 1) password
  - 2) password_confirmation
  - these are *virtual* attributes which are temporary and do not get persisted to the database
- start with 'respond_to' tests and update test model
- add test to enforce presence of password
- add test when pw and pw confirmation do not match
- adding **has_secure_password** to end of User model allows all tests to pass
- for now, comment out this line to get passes to fail
- *version of bcrypt listed '3.1.2' does not seem to load*

### 6.3.3 | User authentication
- final step is to auth a user
  - first, find user by email (acting user name)
      - `user = User.find_by(email: email)`
  - second, auth the user based on given password
      - `current_user = user.authenticate(password)`
      - if pw matches, returns uers; otherwise returns false
      - will implement in ch 8
- remaining tests implemented by **has_secure_password**, uncomment to get green
- add rspec test to check if user responds to **:authenticate**
  - **before** block should have the user to db
  - use **let** method to fetch user from db by searching for email
  - test that user is returned when password is valid
  - test that false isreturn when password is not valid
  - **specify** can be used interchangibly with **it** and should be used when **it** would sound awkward
- using **let**
  - allows for local variables inside tests
  - syntax is odd, but similar function to variable assignment
  - take arg of symbol, and block whose return value is assigned to local variable with the symbol's name
  - *memoizes* value (stores value between invocations)
- add test for password length
- at this point, with **has_secure_password** uncommented, all tests but the length validation should pass
- in block describing auth method, let only hits the db once

### 6.3.4 | User has secure password
- with Rails 3.0 and before, auth system was rolled from scratch; now bundled with lastest version of Rails
- will only need a few lines of code to complete the system
- add validation for min pw length (min 6) to User class
- precense validation for password attribute is added automatically by **has_secure_password**
- second, we need to do the following (provided by **has_secure_password**):
  - add *password* and *password_confirmation* attributes
  - require presence
  - require that they match
  - add **authenticate** method to compare encrypted pw to **password_digest**
- to inspect source code for this *secure_password.rb*
  - automagically creates **password_confirmation** and validation for **password_digest** attribute
- all tests should be green
- if you get warnings regarding *I18n.enforce_available_locales*
  - find in 'config/application.rb' and change config setting

### 6.3.5 | Creating a user
- basic user model is complete
- create a user via Rails Console since we cannot sign in via web (ch 7)
- start rails console w/o sandbox:
    User.create(name: "The Dude", email: "dude@abides.com", password: "foobar", password_confirmation: "foobar")
- value in **password_digest** column is encrypted version of "foobar"
- can test authenticate method:

    >> user.authenticate("invalid")
    => false
    >> user.authenticate("foobar")
    => #<User ... >


## 6.4 | Conclusion
- in this chapter
  - created user model with name, email and various password attrs
  - several validations for important constraints
  - spec testing
  - ability to auth users based on comparision of encrypted pw input
  - leverage **validates** method to add model validations
  - leverage **has_secure_password** for user pw management and auth
- this will provide basic Rails infrastructure for user sign up
- merge back into master

    $ <add and commit to local>
    $ git checkout master
    $ git merge modeling-users

## 6.5 | Exercises
1) add test as given
2) change in user model *before_save* works, '!' denotes to change field
3) add email address with '..' to spec test, should fail.
  change regex from

    /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i

  to

    /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i

  This restricts what comes between the '@' and the final '.'. Instead of at least one char, digit, '-' or '.', you need at least one char, digit or '-', then zero or more ('.' followed at least one char).

4) Read Rails API for **ActiveRecord::Base**          #TODO
5) Read Rails API for *validates*                     #TODO
6) Spend a couple of hours playing with Rublar        #TODO



# Chapter 7 | Sign up #############################################################################

#### Topics:
- showing users
  - gravatar
- signup form
- sign up failure
- signup success

#### References:
- Railscast for custom Rails environments: http://railscasts.com/episodes/72-adding-an-environment
- Factory Girl: http://github.com/thoughtbot/factory_girl
- Gravatar: http://gravatar.com/
- CarrierWave: https://github.com/jnicklas/carrierwave
- Paperclip: https://github.com/thoughtbot/paperclip
- MD5 hash: http://en.wikipedia.org/wiki/MD5
- CSRF attack: http://stackoverflow.com/questions/941594/understand-rails-authenticity-token
- SSL/TLS: http://en.wikipedia.org/wiki/Transport_Layer_Security
- SSL endpoints: https://devcenter.heroku.com/articles/ssl-endpoint

#############################################################################


- user will signup via HTML *form submit*
- once user successfully signs up, show profile page (show user)
  - first step in REST arch for users
- to make profile page, we need a user in db (chicken and egg), but we added one via console in previous chapter
- create new branch

    $ git checkout -b sign-up

## 7.1 | Showing users
- end goal is to show user profile
  - profile picture
  - basic user data
  - list of microposts
  - number following
  - number of followers

### 7.1.1 | Debug and Rails environments
- using conditional check on environment and *debug* method, print params in **application.html.erb**
  - only render when in development env
  - this is a default Rails env, can add more
  - also add basic styling to **custom.css.scss**
  - this styling will include a mixin for box sizing
- *debug* method prints useful info (controller, action, ...) in YAML format
- Rails environments
  - 3 default environments: *test*, *development*, *production*
  - Rails provides **Rails** object with *env* attributes, which has boolean methods to check env
    - e.g. `Rails.env.test?`, `Rails.env.development?`, `Rails.env.production?`
  - to run console in another env, pass it as an arg
    - e.g. `$ rails console test`
  - to start rails server in another env, pass '--environment' flag along with env
    - e.g. `$ rails server --environment production`
  - to run rake commands in another env, need to set the **RAILS_ENV** var
    - e.g. `$ bundle exec rake db:migrate RAILS_ENV=production`
  - these three different ways to specify env are not interchangable
  - env info is also available in heroku console (which should be production)

### 7.1.2 | A user resource
- will represent User data as *resources* (REST) which can be created, shown, updated, and destroyed
- REST
  - data as resources

  | action     | http method |
  |------------| ------------|
  | create     | POST        |
  | show       | GET         |
  | update     | PATCH/POST  |
  | destroy    | DELETE      |

  - resources are referred to by resource name and unique identifier
- for users, User is the resource, and the id is the identifier
  - e.g. url for user with id=1: `.../users/1`
  - right now, get a routing error
- when Rail's REST features are active, GET requests are routed to **show** action
- to get REST style URLs to work, declare resources in **config/routes.rb**;

  SampleApp::Application.routes.draw do
    resources :users
    root  'static_pages#home'
    ...

- this adds all RESTful urls for users resource
  - Table 7.1 has all mappings
- now error for missing 'show' action in controller
- add new view at 'app/views/users/show.html.erb' and fill in to display user name and email
  - assume existence of user instance variable
- define show action in users_controller
  - define @user
  - use **User.find** method, passing in the id to get the user from the databse
  - getting id from params hash returns string (always), **find** converts to int
  - view should render now
  - will throw error if no user exists with that id

### 7.1.3 | Testing the show user page
- using integration tests for pages associated with Users resource
- update 'spec/user_pages_spec.rb'
  - add spec test for profile page
  - use *user_path* named route, passing it a User model object
  - write basic test for user name in content and title
- could use ActiveRecord **User.create** to create our model object, but factories are more convenient
- add *Factory Girl* gem to project, in **:test** group: `gem 'factory_girl_rails', '4.2.1'`
  - TODO check version
- Factory Girl (from thoughtbot)
  - defines DSL
  - specialized for defining ActiveRecord objects
  - syntax: Ruby blocks and custom methods
  - advanced features will be leveraged in later chapters
- all factories in **spec/factories.rb** (auto loaded by rspec)
  - add code for User factory
  - passing ':user' symbol tells FactoryGirl definition for User model object
- all specs should be green (may need to bounce spork)
- notice that specs are slow
  - not Factory Girl's fault!
  - bcrypt is slow, which makes it harder to attack
  - this tradeoff between speed and security is ideal in production, but not testing
  - update **config/environments/test.rb** to change *cost factor*
    - this will increase speed, but reduce security (only in test env)

### 7.1.4 | A Gravatar image and a sidebar
- for view development, less focus on structure/testing
- will resume TDD for pagination
- start with adding Globally Recongnizatble Avatar (Gravatar)
  - free service to associate info + image with email address/account
  - this way we do not have to manage our own images
  - resources if you need to manage images
    - CarrierWave (https://github.com/jnicklas/carrierwave)
    - Paperclip (https://github.com/thoughtbot/paperclip)
- define helper function **gravatar_for** which takes a user and returns a Gravatar image
  - add line to view (tests fail due to template error)
- create file for helpers associated with **user_controller**
- Gravatar URLs are based on a MD5 hash of the user's email
  - implemented by **hexdigest** method in the **Digest** library
  - e.g. `Digest::MD5::hexdigest("some.email@inter.net".downcase)`
  - NOT case insensitive, like email
- define helper method **gravatar_for** in **app/helpers/users_helper.rb**
  - construct url using user email and MD5 hash
  - generate image tag with url with class "gravatar"
  - specs should pass
  - TODO verify it works (no internet access atm)
- default Gravatar image is shown when email does not map to a valid email
- set example user to 'example@railsturtorial.com' for a valid example account (Rails logo!)
  - will use *aside* HTML element along with Bootstrap classes *row* and *span4* for sidebar
  - update **app/views/users/show.html.erb**
- update **custom.css.scss** for Gravatar and sidebar styling


## 7.2 | Signup form
- goal of this section is to create a sign up page with four fields: name, email, pw, and pwconf
- clean/reset database
  - remove data: `bundle exec db:reset`
  - prepare db: `bundle exec test:prepare`
  - bounce local server

### 7.2.1 | Test for user signup
- use rspec and capybara to verify valid and invalid input to the signup form
- can enter test into input fields with **fill_in** method
- can click butons with **click_button** method
- to verify valid and invalid input during tests, use ActiveRecord's **count** method
  - e.g. `User.count`  => 0
  - when invalid info is submitted, count should not change
  - when valid info is subimttied, count +1
  - **change** method used to calc count before/after execution of *expects* body
- in Rspec, **eq** = equals
- remember to factor out common code into memorized local (submit button)
- put code run in every case in *before* block
- at this point, fails due to lack of submit button
- to get these basic tests to pass:
  - create expected elements on page
  - route submit button to correct endpoint
  - create user in db, only if data is valid

### 7.2.2 | Using *form_for*
- **form_for** is an ActiveRecord helper method which takes an ActiveRecord obj and creates a form from it's attributes
- in Rails 2.x and before, used `<% form_for ... %>`; later versions use `<%= form_for ... %>`
- form_for
  - takes a block with one variable
  - uses variable to call methods to generate HTML form elements (text_input, label, radio button, submit, password field)
  - generate code to set attr of obj passed in
  - symbol passed to these methods correspond to attributes of the Active Record obj
- currently page breaks and tests fail since **@user** is nil
  - rspec: can pass '-e' flag which takes string and matches test with matching description string
  - (?) claims this is different than substring matching, but cannot reproduce
- to fix, we need to create a new User obj in the **user** controller, **new** action
  - e.g. `@user = User.new`
  - some specs still fail since *subit* does not map to an action
- update styling in **custom.css.scss**

### 7.2.3 | The form HTML
- ids of HTML input elements are "#{obj}_#{attr}"
- Rails generates auth token to prevent *cross-site request forgery (CSRF)*
- **text_field** generates input element with type "text"
- **password_field** generates input element with type "password"
- **label** generates label element with ref to input(?)
- **name** field is set on each HTML element, which allows RAILS to construct the hash
- *form* tag is also generated, Rails populates with info about class, URL, an id, and a HTTP method
- form and tests are still broken since user **create** method is not defined


## 7.3 | Signup failure
- in this section, update form to accept invalid input, re-render page with list of errors

### 7.3.1 | A working form
- RESTful routing provided by `resources :users` maps *POST* requests to '/users' to **create** action
- **create** action should create new User obj with **User.new** and save to db
  - this section is concerned with implementing the failure to save and re-rendering page
  - use result of **@user.save** to switch success/failure
- now clicking the submit button doesn't work, but we can get some useful information about what is submitted
  - params for this request is a hash of hashes
  - keys within the *user* hash correspond to the names of the input tags
- hash keys are passed to Users controller as symbols
  - **params[:user]** is the hash of attrs we need for **User.new**
  - worked in the past but was error prone and insecure by default
  - as of Rails 4, this no longer works

### 7.3.2 | Strong parameters
- mass assignment as mentioned is convenient, but is dangerous and insecure since all user input is passed
- previous versions of Rails used a method **attr_accessible** in the model layer to solve this
- in Rails 4, preferred method is *strong parameters* in the controller layer
  - allows us to specify which parameters are *required* and which are *permitted*
- default behaviour does not allow mass assignment, so more secure by default
- restrictures on params
  - require: user
  - permit: name, email, password, password_confirmation
  - add private helper to access params (convention)
  - pass output of this helper to **User.new**
- only 1 spec failture! (since we cannot successfully save a user)

### 7.3.3 | Signup error messages
- Rails provides error messages based on model validations
  - **obj.errors.full_messages** contains array of all error messages
- add a partial to be rendered when there are error messages
  - spec test as exercise; this is not the final version
  - checks if errors exists
  - leverage **count**, **any?** (somplement of **empty?**) methods
    - work on objs, arrays, strings
  - also use text helper **pluralize** (not available by default in console)
    - takes an int and some string
    - will pluralize string using underlying *inflector*
- add styling to **custom.css.scss**
  - uses SASS **@extend** to include functionality of two bootstrap classes


## 7.4 | Signup success
- if save is successful, route to user profile with message

### 7.4.1 | The finished signup form
- tests fail b/c there is no template for the **create** action (and we wont make one)
- will use **redirect_to @user** for signup sucess
  - test should be passing
- add flash to greet new user

### 7.4.2 | The flash
- flash will show when user successfully signs up
  - disappears on page reload or when visiting any subsequent page
- Rails has special variable *flash* which can be treated like a hash
- add it to application layout
  - iterate through each entry in flash hash
  - use key to set div class
  - use value to set flash message
- notes that keys are symboles, but erb converts to string automatically
- set flash in user controller, **create** action
- flash is NOT time senstive (no timeout)

### 7.4.3 | The first signup
- create user with name 'Rails Tutorial' and email 'example@railstutorial.org'
  - should route to user page
  - should see success flash
- verify via rails console `User.find_by(email: 'example@railstutorial.org')`

### 7.4.4 | Deploying to production with SSL
- make sure Heroku is set up (see ch 3)
- will use Secure Sockets Layer (SSL), which is technically Transport Layer Security (TLS)
- ensures that signup is secure and immune to *session hijacking*
- merge changes to master branch
- add line to production.rb to force SSL in this env
- push to heroku: `git push heroku master`
- run migration on heroku: `heroku run rake db:migrate`
- if we were building our own site (say example.com), we would need to purchse ssl cert
- we can piggyback on heroku's!
- wah hoo!

## 7.5 | Conclusion
- build basics for user signup
- ch 8 will tackle sign in/out
- ch 9 and beyond fill in REST for user model, info update and security

## 7.6 | Exercises
1) add optional size param to **gravatar_for** helper
2) Write test to check that correct page is loaded, correct number of errors are shown, and flash is present (count 2)
3) verify tests for testing user creation
4) user **content_tag** to clean up flash code and verify tests still work


# Chapter 8 | Sign in, Sign out #############################################################################

#### Topics:
  - sessions
  - sign in
  - sign out
  - cucumber

#### References:
- Gherkin: plain text language used by Cucumber [https://github.com/cucumber/gherkin]

#############################################################################

- change layout based on signin status
- restrict access to pages based on identity
- investigate cucumber for Behavior Driven Development (BDD)
- create new branch `$ git checkout -b ch8-sign-in-out`

## 8.1 | Sessions and signin failure
- a *session* is a semi-permanent connection between two computers
- we will use sessions for handling *signing-in*
  - forgetting session on browswer close
  - providing checkbox for persistent sessions
  - remembering sessions until user signs out
  - * also common to expire after some timeout; not used here
- model sessions as a restful resource
  - sign-in = create
  - sign-out = destroy
  - will use *cookies* instead of db
- in this section
  - sessions controller + actions
  - signin form

### 8.1.1 | Sessions controller
- handled by new action
- send **POST** request to create action
- send **DELETE** request destroy action
- to generate session controller and integration test

    $ rails generate controller Sessions --no-test-framework
    $ rails generate integration_test authentication_pages

- signin page at **signin_path**
- start with minimal spec for auth page
- add **resources** to route file (only new, create, destroy) and matching routes

    resources :sessions, only: [:new, :create, :destroy]
    match '/signin',  to: 'sessions#new',           via: 'get'
    match '/signout', to: 'sessions#destory',       via: 'delete'

- **via: 'delete'** should be invoked via HTTP delete

  | HTTP Method | URL         | Named Route    | Action   | Purpose              |
  |------------ | ------------|----------------|----------|----------------------|
  | GET         | /signin     | signin_path    | new      | page for new session |
  | POST        | /sessions   | sessions_path  | create   | create new session   |
  | DELETE      | /signout    | signout_path   | destroy  | delete session       |

- to see all routes, run **rake routes**
- update controller with aforementioned actions
- create signing page **app/views/sessions/new.html.erb**
- NOTE: test for title is case senstive, test for content is not

### 8.1.2 | Signin tests
- similar to signup form, except only two fields
- when signin is invalid, rerender a signin page and flash error
- Capybara 2.0, **have_selector** only selects visible HTML elements
- add spec test for click sign in button with invalide (none) information
- to test for successful signin, we'll check
  - title (which should reflect user's name)
  - changes to navigation
    - a link to profile page
    - a sign out link
    - lack of sign in link
- defer testing 'Settings' and 'Users' until ch 9
- write tests for valid info
  - **have_link** which matches text and href
  - use upcase on email to ensure match

### 8.1.3 | Signin form
- will use **form_for** again
- no Session model, so wont be exactly the same as User signup
- Rails can refer to **action** of the form
  - need to give name of resouce and url to point to
- could also use **form_tag** instead of **form_for**, but not common in signin pages
- generated form HTML very similary to signup form

### 8.1.4 | Reviewing from submission
- params has *session* which is a hash containing submitted *email* and *password*
- populate **create** action in the sessions controller to find user by email and auth pw
  - recall email is save in downcase for case insentivity
  - **authenticate** method (provided by **has_secture_password**) returns false if incorrect pw
  - check for existence of user and auth pw (common pattern)
    - auth only checked if user is anything but *nil* or *false*

### 8.1.5 | Rendering with flash message
- cannot use same technique to render flash messages, bc session is not an Active Record object
- naaive approach is to assign value to flash obj in controller, weird results
  - flash persists for one request
  - render does not count as a request, like redirect
  - therefore, flash will persist for one request longer than we want
  - e.g. submit bad info, then go to homepage; flash persists!
- add test to catch this
  - add nested test which will click home link, and check again for the flash
- will use **flash.now** instead of **flash**
  - used for displaying flash on rendered pages
  - disappears on additional request
- tests are now green, wahh hoo

## 8.2 | Signin success
- most ruby intensive section
- fill out create action in sessions controller
  - **sign_in** function (helper to implemenet)
  - redirect to user profile page

### 8.2.1 | Remember me
- will remember session forever and only sign out user when user expliclty does so
- signin functions will cross MVC lines
- could write new class, but Ruby provides modules so we will use *SessionsHelper* module instead
  - this module is available in the views rendered by Sessions controller
  - not available in controllers
    - add include to application controller: `include SessionHelper`
- HTTP is stateless, therefore web app must track user progress from page to page
- Rails has build in **session** function which is used to store *remember_token*
  - setting this token to user id make its available to each page
  - stores in browswer cookie, expires on browser close
  - secures sessions since spoofed ids wont match rememebred token
- we want sessions to persists even after browswer close
- need a permantent way to id signed in user
- will generate remember token for each user
  - store as permanent cookie which does not expire on broswer close
- since remember token is associated with the user, add it to the User model
  - remember, tests first!
  - generate migration: `rails generate migration add_remember_token_to_users`
  - fill in migration with string col for token and index (since we will look up users with it)
  - does not have to be unique?
  - update the database!
- specs are green
- what will the remember token be?
  - large random string which is unique
  - will use **urlsafe_base64** method from **SecureRandom** module in Ruby std lib
    - will return string of 16 chars
    - can be chars, digits, dash, and underscore
    - prob of collision low: 1/(64^16) = 10^-29
  - will store this in browser and an encrypted version in the database
- will get cookie from browser, encrypt, and search for user
- encrypt in db is good, if compromised, cannot be used to sign in
- will change this every time a new session is started
  - mitigate affects of *session hijacking*
  - site wide SSL so that tokens are not visible over public wifi
- will use a *callback* in the context of email uniqueness to verify that every user has a valid token
  - will use **before_create** to create token for newly created users
- to tests, save a test user for the first time and check that token is not blank
  - this test will intro **its** which looks at the given attribute and not the subject
- add **before_create** filter to user.rb
  - pass it a *method reference* instead of a block, like we did in **before_save**
  - define private method **create_remember_token**
  - method will assign an attr of user using **self**, which referes to user's remember_token
  - also create methods to create and encrypt token
- private methods are hidden from the interface: `User.private_method` doesn't work
  - 'to_s' is to handle the nil case (might happen in tests, but shouldn't happen in browser)
- using SHA1 hashing algo to encrypt
  - much faster then bcrypt
  - speed is important bc this will be run on each page
  - less secure than bcrypt
  - encryption of a 16 char random string essentially uncrackable
- **encrypt** and **new_remember_token** are attached to the **User** class
  - this way they do not need an instance of **User** class to be called
    - aka class method, since it needs no instance
  - declared public so they can be used outside of the User class
- user_spec.rb should now be passing
  - NOTE: encrypt method will return hash for nil and ''!

### 8.2.2 | A working sign_in method
- steps for sign_in method

1) create new token
2) place unencrypted cookie in browser using Rails **cookies** utility
3) save encrypted cookie in database using **update_attribute**
4) set *current_user* to *user*

- step 4) isn't necessary due to redirect from **create** action, however not good to rely on this
- **update_attribute** allows update of single attribute while bypassing valdation
  - in our case, we would need the password and confirmation to pass validation
- **cookies** Rails utility allows manipulation of browser cookies as if they were hashes
  - are not hashes, it's an abstraction
  - each element takes a hash of two elements
    - **value** of the cookie
    - **expires** date (optional)
  - has **permanent** method which sets **expires** to 20 years
- intro to *Rails time helpers* on fixnum
  - creates useful strings from semantic method chains
  - e.g. `1.year.from_now` prints the date a year ago
- can now fetch users using remember token: `User.find_by(remember_token: remember_token)`

### 8.2.3 | Current user
- need a way to retrieve user on subsequent page views
- **current_user** will be available in the controller and the view
- define *assignment function** (special syntax)
  - takes arg, user, which is right hand side of assignment
  - sets user to instance variable @current_user, storing for later user
- can also define a *getter*, but this is the same as **attr_accessor**, which defines setters/getters
- this simple implementation will not persist @current_user between page views
  - due to stateless nature of http interactions
  - subsquent requests sets variables to defaults, such as @current_user = nil
- use the **User.encrypt** method with the browser cookie to fetch the current user
  - can memoize lookup using the *or equals* operator ( ||= )
  - memoize is only effective if **current_user** is used more than once per page view
- *or equals*
  - if value on LHS is nil or false (falsey in Ruby), then assigns RHS
  - else returns LHS
  - shorthand for: `x = x || y`
  - *or* evals from left to right, *shortcircuit evaluation*
  - follows similar for as `x = x + y === x += y`

### 8.2.4 | Changing the layout links
- update the header to show "Sign in" when the user is not signed in and "Sign out" when signed in
  - will also add Accounts menu with links to "Profile", "Settings" and "Sign out"
- implement a **signed_in?** method
  - checks if **current_user** is non-**nil**
  - uses the "not" operator, **!** (bang)
- Add links to dropdown using bookstrap dropdowns
  - "Sign out" link to user HTTP DELETE, needs to be specified
  - browsers cannot issues HTTP DELETE, Rails fakes it with js
- "Profile" link uses **current_user** as path
  - Rails allows us to refer to user
  - auto converts **current_user** to **user_path(current_user)**
- add Bootstrap js to **application.js**

  `//=require bootstrap` in *application.js*

- signed in user now sees 'Account' dropdown menu
- verify we can signin, broswer has cookie for *remember_token*, and specs are green
- signout does not work, since sessions controller does not have destroy

### 8.2.5 | Signin upon signup
- current implementation, user is not signed in by default when signing up
- add test to 'after user saved' to check for 'Sign out' link
  - this is a side effect of signing in
  - this check is also performed in *authentication_pages_spec.rb*
- update **users_controller create** method
  - if **@user.save** is true, call **sign_in** method passing it **@user**
  - **sign_in** action is defined in **sessions_helper.rb**

### 8.2.6 | Signing out
- use RESTful convention to signing out
  - sign in used **new** for the page and **create** to complete signin
  - sign out will use **destroy** action to delete session
- add test nested within sign in w/ valid info
  - click 'Sign out' link
  - verify that 'Sign in' link is now present
  - optionally check that 'Sign out' link is gone (redundant if we assume mutual exclusion)
- in destroy action
  - sign user out by calling **sign_out**
  - send user to home by calling **redirect_to root_url**
- **sign_out** method
  - defined in Sessions helper module
  - change current user's token in db
    - in case cookie was stolen, it is now invalid
  - user **cookies.delete** to delete the **:remember_token** cookie
  - unset the current user (assign to nil)
    - this happens as a side effect of the **redirect** but it is not good to rely on this
- all specs green
  - important to not this does not test everything, i.e.
    - how long "remember me" is set
    - if cookie is set and deleted
  - author notes that these kind of tests tend to rely on implementation which are brittle and can change
  - these test for core functionality: sign in, stay on the page, sign out

## 8.3 | Introduction to Cucumber
- Behavior Driven Development (BDD) stlye testing framework
- allows for plain text stories to define tests
  - can be shared with non-tech people
  - readabiliy
  - can be verbose
  - emphasis on high level behavior instead of low leve implementation

### 8.3.1 | Installation and setup
- install the following gems
  - cucumber-rails
  - database_cleaner
- run: `rails generate cucumber:install`
  - will create **features/** directory

### 8.3.2 | Features and steps
- tests in **features/** director with *.feature* extension
- cucumber uses plain text language, Gherkin
- tests are specified by:
  - Features, which have ...
    - Scenarios, which have ...
      - Given statements : setup
      - When statements : context
      - Then statements : condition
- Given, When and Then statements are plain text in test, but defined in *step files*
  - map plain text to ruby code (capybara)
  - **features/step_definitions**
  - *.rb* files
  - these are methods which takes regex and block
    - regex matches text of statement in *.feature* file
    - block is Rspec/Capycara code to execute
- the Rspec code is very similary to our code from previous spec tests
  - access to **page** object
- run tests
  - `bundle exec cucumber features`
  - `bundle exec rake cucumber`
  - `bundle exec rake cucumber:ok`

### 8.3.3 | Counterpoint: Rspec custom matchers
- case: integration tests vs. Cucumber
- cucumber seperates concerns
  - tests are seperate from code that implements them
    - if implementation changes, tests can stay the same, only steps change
  - this is both good and bad
  - offers highest level of abstraction
- cucumber: easy to read, harder to write
- rspec: harder to read, easier to write
- Rspec *custom matchers* and helper methods
  - keeps code DRY
  - seperate concerns like cucumber
- add custom matcher for 'have_error_message' and to fill in valid user info
  - define in same utilities file as **full_title_helper**
    - note: **full_title** is actually defined in applicaiton helper which is included

## 8.4 | Conclusion
- implemented full suite of registration and login behaviors
- next step to restrict access to pages based on user idenity and sign on status, ch 9
- also in ch 9
  - user to edit info
  - admin to remove users
- merge changes to back to master
- push to Heroku

## 8.5 | Exercises
1) refactor sign in form to use **form_tag** instead of **form_for**

- http://railscasts.com/episodes/270-authentication-in-rails-3-1
  - **http_basic_authenticate_with** :name => 'something', :password => 'something'
- **form_for** is used when form is backed by specific model
- **form_tag** creates basic form
  - do not def tags based on form
    - f.text_field -> text_field_tag
  - params hash no longer has form object layer
    - params[:session][:email] -> params[:email]

2) refactor tests to leverage helper funcitons and rspec custom matchers, also refactor into different files

- rpsec custom matchers must return one value?
  - returns one value true/false
  - how to combine multiple?
  - cannot use negative 'not_to'? -> didn't work in case of signin/sing out

# Chapter 9 | Updating, showing, and deleting users #############################################################################

#### Topics:
- updating users
- authorization
- showing all users
- deleting users


#### References:
Clearance (Thoughtbot) : https://github.com/thoughtbot/clearance
Faker Gem : 
will_paginate/bootstrap-will_paginate Gems : 
DELETE w/o js : http://railscasts.com/episodes/77-destroy-without-javascript

#############################################################################

- update Users resource
  - edit, update, index, destroy
- enforce security model
- sample data
- pagniation
- destory users
- priviliged admins

## 9.1 | Updating users
- editing user follows similiar pattern to creating user (ch 7)
  - **new** action to render view for new users -> **edit** action to render view
  - **create** responds to POST request -> **update** responds to PATCH request
- signed in user can only edit their own info
  - need to intro access restriction
  - will leverage auth infrastructure introduced in ch 8 and *before_filter*

### 9.1.1 | Edit form
- test for this page are analogous to those for sign in page
  - create test user
  - visit page
  - test for correct page
    - test for title
    - test for content
    - test for change link
  - test invalid input
    - click submit
    - test for error content
- fill out **edit** action in *Users* controller
  - proper url is 'users/:id/edit'
  - *id* is available in params hash: `params[:id]`
  - find user by id and assign to instance variable
- fill out the 'users/edit' view to get tests to pass
  - reuse 'errors' partial
  - form is same as /users/new
  - passing **@user** to form, Rails prepopulates *name* and *email* fields
- form generated HTML
  - hidden input field with name **_method** and value **patch**
  - broswers cannot submit PATCH request, as needed by REST
  - broswer users POST request
  - this is how Rails fakes it
- form_for uses POST or PATCH depending if the record is new
  - uses **new_record?** method to check if record is new
  - true: uses POST to **create** new user
  - false: uses PATCH to **update** existing user
- add 'Settings' link to drop down menu
  - add test to auth specs
  - fill in header view
- fill in **sign_in** helper method to sign in user with valid info
  - provide option to manipulate cookie if not using capybara
  - necessary when using HTTP requests (**get**,**post**,**patch**,**delete**) directly

### 9.1.2 | Unsuccessful edits
- define **update** method
  - attempt to update user attributes `user.update_attributes(user_params)`
  - re-use *user_params* method (strong parameters)
    - *strong parameters protect against mass assignment vulnerability*
      - [wikipedia](http://en.wikipedia.org/wiki/Mass_assignment_vulnerability)
- if invalid info, then **update_attributes** will return false
  - render 'edit' page
  - add **flash.now[:error]**
- all tests are green

### 9.1.3 | Successful edits
- image edit done with gravatar
- tests
  - define new name and email
  - fill in form with new info and current password
  - click save
  - verify that profile page shows
  - verify success flash
  - verify user is still signed in
  - verify that user name and email have been updated
- need to use **reload** method to reload user info from db
- fill in **update** action for successfuly model update
  - flash success method
  - redirect to user instance (profile page)
  - note, the redirect takes care of the flash.now vs flash issue
- edits require password with every update, annoying but more secure
- all specs pass
- functionality complete


## 9.2 | Authorization
- *authentication* allows us to identify users, *authorization* allows us to control what they do
- edit and update are funcitonally complete, bit security flaws
- signed in users can update ANY user's info
- non-signed in users can access edit and update
- will prevent signed in users from updating anyones info but they're own
- will redirect non-signed in users to sign in page (plus helpful message)

### 9.2.1 | Requiring signed in users
- restrict access to **edit** and **update** actions
- users who are not signed in, redirect to sign in page
- user *patch* request to access **update** action
  - no way to access via browser directly
- when user HTTP request directly, have access to *response* obj
- user *before filter*, **before_action**
  - defaults for all actions
  - use :only filter to restrict to certain actions
  - in our case **edit** and **update**
- redirect offers shortcut for populating **:notice** flash
  - assigned via options hash
  - does not work for **:error** and **:success**

### 9.2.2 | Requiring the right user
- ensure that correct user is accessing **edit** and **update**
- will do this with another before filter which checks for the correct user
- will not use capybara for tests
  - access response objects directly
  - submit get and patch requests to edit and update, respectively
- since users should not be attempting this, redirect to root url, not signin
- can pass options hash into factory girl, which overrides default attr values
- before action, **correct_user**
  - declare instance variable, using lookup with id from params hash
  - verify if current_user?
  - potential redirect to root url
- add **current_user?** to Sessions helper
  - does comparison of current user and user in arg
- we can remove the declaration of this instance var in **edit** and **update** actions

### 9.2.3 | Friendly forwarding
- if non-signed in user tries to access restricted page (like edit)
  - redirects to signin page
  - after successful signin, redirects to root
  - would like to make it redirect to original protected page
  - this may enevitably send the user to root if it's not their page
- testing
  - create test user
  - visit edit path for that user
  - *redirects to signin page
  - enter valid credentials
  - click signin
  - ensure you're on the edit page for that user
- in order to accomplish this, need to store intended location
- we will store this in the session variable (similar to cookies variable, provided by Rails)
- in Sessions Helper
  - **store_location**
    - stash request url in session variable
    - only return url for GET requests
      - otherwise GET request would get sent to a url expecting PUT, PATCH, DELETE; BAD!
  - **redirect_back_or**
    - redirects to stored url or default, then deletes stored url
- call **store_location** in before filter, before redirect to "please sign in"
- call **redirect_back_or** when creating a session, passing **user** as the default

## 9.3 | Showing all users
- will fill in **index** action for User, which shows all users
- seed db with data
- paginate user output

### 9.3.1 | User index
- **show** pages available to all users (signed in and not)
- **index** or all user page only available to signed in users
- write test to ensure that non-signed in user cannot access **users_path**
  - part of auth tests
- in controller
  - define **index** method
  - add this actiont to before filter **signed_in_user**
- write tests for signed in user
  - user factory to create three unique users
  - sign in with first users
  - visit **users_path**
  - make sure page has 'All users' content and title (correct page)
  - make sure that all users created are shown within *li* elements
  - in user pages spec
  - NOTE: if subject is set to page, then custom matchers, which expect page, do not seem to work (page undef)
- back to users controller
  - fill in **index** action, assigning **User.all** to an instance var
  - this is a bad idea in general (i.e. a lot of users), will revisit when paginating
- create *index* view
  - provide title and h1
  - create an unordered list
  - fill *ul* with *li*s
    - each list item has gravatar image (using helper) and link to profile page with user name as text
- add some SCSS
- add url to **users_path** to header for signed in users
- all green and it works ... but it's a ghost town up in here

### 9.3.2 | Sample users
- will create rake task to populate the db with example users
- will use Faker gem to create names
- rake task:
  - defined by *namespace* (in our case **db**)
  - creates 1 sample users and 99 fakes
  - pass the environment to the task so it has access to db
  - user **User.create!** which throws on invalid input (instead of just returning false)
    - noisier
  - run the following commands to clean, seed and rebuild db

      bundle exex rake db:reset       # clear db
      bundle exex rake db:populate    # this is our task!
      bundle exex rake test:prepare   # rebuild test db

### 9.3.3 | Pagination
- too many (all) results returned on index page
- will use *will_paginate* and *bootstrap-will_paginate* gems for pagination and styling
- write tests for index page
  - check for div with pagination class (will not test gem itself)
  - verify that all users are present (pagination only, not all users)
- we will need to create a lot of users at once -> FactoryGirl sequences
  - sequence takes a symbol for the corresponding attr and a block with one var
    - e.g. `sequence(:name) { |n| "Person_#{n}" }`
  - this allows us to assign an index with each sample user (i.e. "Person_1", "Person_2", etc.)
  - block var is automatically incremented
- will use **before(:all)** in spec tests for pagination
  - passing the **:all** symbol as an arg tells rspec to run this *once* before all tests in block
  - performance since recreating would be redundant
  - complimentary **after(:all)** to delete db entries
 - in **Users controller index** action
  - update to return only one page of users instead of all
  - take page out of params
  - defaults to page 1 (will break if "", added default yourself)
- in **user/index** view
  - call the **will_paginate** before and after the **ul** containing the list of users
    - automatically finds **@users** (since we in the users/index view)
    - adds markup to paginate through users
    - nil defaults to first page
    - expects users to be fetched using the **paginate** method

### 9.3.4 | Partial refactoring
- in **index** partial, we can call **render** on the *user* variable directly
- Rails knowns to look for a partial with the name **_user**
- can even call **render** on the collection of users, **@users**
- Rails will iterate through collection and call render on the **_user** partial for each
- refactoring should not break spec tests, which test functionaliy


## 9.4 | Deleting Users
- **destroy** is the last REST action left
- will define **destroy** action to delete users
- will first create class of user, *admin*, who are the only ones authorized to do so

### 9.4.1 | Administrative users
- will add a boolean column to user table to designate admins
  - will automatically create **admin?** boolean method to test if a user is an admin
- write tests on user model
  - should respond to **admin**
  - admin should be false by default
  - can toggle to true (**user.toggle!(:admin)**)
- rspec convention `it {should be_admin}`, implies **admin?** method exists
- write migration for adding this column to db, setting default to false
  - default not necessarily needed, since default is **nil**, which is false
  - this is more explicit, which is good
- run migration and rebuild test db
- update **populate** rake task to build 1 admin user and 99 non-admins
- Revisiting strong parameters
  - without protection, anyone can submit a patch request with admin=true
  - huge security problem
  - essential to pass only safe-to-edit params
  - do not include admin here, this will deny web requests to update this attr
  - good idea to write a tests for this, but left as an exercise

### 9.4.2 | the `destory` action
- need to add destroy links and implement destroy action
- only add destroy links to index page for admins and not their own profile
- add admin block to user factory to set admin to true
  - create admin: `FactoryGirl.create(:admin)`
- ordinary users should not see delete links
- tests
  - ordinary users should not see delete links
  - admin users should see delete links
  - admin should not see a delete link for themselves
  - clicking any delete link should reduce count by 1
- tests use **match: :first** which tells Capybara to use first match
- update the view
  - add condition to check for admin
  - if admin show link for delete which uses **method: :delete**
- remember, the browser cannot issue delete request so Rails fakes it in js
  - this means that DELETE will not work w/o javascript enabled
  - can fake it with a POST and a form
  - see Railscast for more info
- in controller
  - add **:delete** to **signed_in_user** before filter
  - define destroy method
    - find user by id
    - call destroy on user
    - add a flash success message
    - redirect to users_user
- Problem: any user can issue a DELETE request to delete any other user!
  - this can be done from the command line using CURL
- add to auth page tests
  - if non-admin calls delete
  - expect response to redirect to root url
  - similar to validation for **patch**
- still has flaw that admin can delete themselves, although not through ui
  - they got whats coming to them
- define **admin_user** in before filter in *users controller*
  - redirect to root url unless current user is admin

## 9.5 | Consclusion
- users can now:
  - sign up
  - sign in/out
  - edit their info
  - see user profile pictures
  - see index page of all users
- ch 10: Twitter-like micro posts
- ch 11: status feed of posts from followed users
- these chapters will use **has_many** and **has_many through**
- merge branch, push to heroku

## 9.6 | Exercises
1) make sure updating to admin does not work
  - first add :admin to users_param to allow it to be set (be sure to remove!)
  - in tests
    - create a params hash user: admin, pw, pw_conf
    - sign in user
    - submit a patch request, using the **user_path(user)** for route, and passing in params
    - reload user and make sure that user is NOT admin
  - removing admin from users_params should allow test to pass

2) update 'change' link nexto to Gravatar image to open in a new tab or window
  - use **link_to** helper
  - set target to '_blank'
  - Refs:d
    - http://stackoverflow.com/questions/12133894/open-link-in-new-tab-with-link-to
    - http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html

3) Add test to make sure that Profile and Settings do not appear when not signed in
  - within descript invalide data
    - add check that settings and profile links should not be present
    - refactor checks into a rpsec matcher
    - within matcher, add check for user var and default to {} if not present

4) use **sign_in** in as many places as possible...didn't really find anywhere it was already used
  - 'attempting to access a protected page' could use it, but want to ensure redirect is happening

5) refactor sign up and edit form to use fields partial
  - text on confirmation field needs to be resolved between two pages

6) redirect signed in users to root url when accessing **new** and **create** actions
  - define new test within *user page specs* for the signup page
  - create and signin test user (no capybara option)
  - make sure that get to *signup_path* and post to *users_path* both redirect to root_url
  - add before filter to *users controller* to check for signed in user using **signed_in?** helper

7) learn about the **request** object TOOD
  - http://api.rubyonrails.org/v4.0.0/classes/ActionDispatch/Request.html

8) verify that friendly forwarding forgets intended destination after first log in
  - within test to access protected pages
    - within after sign in test
    - sign out user (use click_link 'Sign out', solution in book is not correct)
    - sign user back in
    - verify that user is brought to profile page and not the edit page

9) modify **destroy** action so that admins cannot delete themselves
  - add a non-capybara spec test to send DELETE request to admin url and check count
  - add logic in destroy action to check if requested user is the current user
  - if so flash error and do not destory user




# Chapter 10 | User Microposts #############################################################################

# Topics:
- micropost model
- showing microposts
- manipulating microposts

# References:



#Notes

- ch9 completed REST actions for users
- this chapter will add the user microposts resource
- associate user and microposts data models using `has_many` and `belongs_to`
- create new branch 'ch10-user-microposts'

## 10.1 |  A Micropost model
- microposts data model will include validations and associations with the user data model
- unlike user model:
  - fully tested
  - default ordering
  - automatic destruction if parent user is destroyed

### 10.1.1 | The basic model
- model only needs two attributes:
  - content
  - user id (to associate user/owner with post)
- generate model: `rails g model Micropost content:string user_id:integer`
- remove generatd factory (will do by hand): `rm -f spec/factories/microposts.rb`
  - mine did not
- goal: get all posts for user in reverse chrono order
  - add index for microposts on user id and created at
  - add to migration: `add_index :microposts, [:user_id, :created_at]`
  - creates *mulitple key index*
  - Active Records uses both keys simultaneously
  - `t.timestamps` creates updated_at and created_at columns
- basic tests for model
  - verify responds to `content` and `user_id` attrs
- to get tests to pass, migrate and prepare db
  - `bundle exec rake db:migrate`
  - `bundle exec rake test:prepare`
- tests should pass, althought not written in "rails way"

### 10.1.2 | The first validation
- need to validate Micropost model
  - user id to indicate which user wrote the post
  - correct way is to use Active Record *associations*
  - for now we will write the test, get it to pass and refactor in the next section
- testing
  - set user id to nil
  - make sure it should be invalid
  - existing case add condition that it should be valid
- tests will be red
  - add validation via *validates* for the presence for user_id in the Micropost model

### 10.1.3 | User/Micropost associations
- each micropost belongs to one user
- each user can have many microposts
- will use *belongs_to* and *has_many* **associations** respectively
- Rails generates the following methods for these assciations

| Method                       | Purpose |
------------------------------------------
|`micropost.user`              | Return the User obj associated with post. |
|`user.microsposts`            | Return an array of Microposts of user's microposts (returns empty array) (returns ActiveRecrod::Asscociations::CollectionProxy) |
|`user.microposts.create(arg)` | Create a microposts and set user_id field to user.id |
|`user.microposts.create!(arg)`| Create a micropost and save to db (?)(exception on failure) |
|`user.microposts.build(arg)`  | Return a new Micropost obj (user_id = user.id) |

- the conanical way to build a micropost is through the user
  - instead of >> use this:
    - Micropost.create  >> user.microposts.create
    - Micropost.create! >> user.microposts.create!
    - Micropost.new     >> user.microposts.build
  - user id is automatically assigned
- update Micropost model spec
  - use build()
  - check that it responds to :user
  - check that user is the equal to the user we created it with
- update user model spec (more to come in next section)
  - check that it responds to :microposts
- update Microposts model
  - `belongs_to :user`
- update User model
  - `has_many :microposts`
- all specs should be green


### 10.1.4 | Micropost refinements
- previously tested just existence of microposts attr on user
- add *ordering* and *dependency*
- verify that `user.microposts` returns an array
- create a micropost factory to *factories.rb* via FactoryGirl

#### default scope
- by default, no guarantees about ordering of `user.microposts`
- testing
  - create two posts
    - one a day ago another an hour ago
    - Factory Girl allows us to set *created_at* attr, which is automatic from Active Record
    - most dbs order by id, so these would be returned in wrong order
    - use `let!` to create variables immediately and set ids
      - `let` variables are lazy, not created until used
  - use `to_a` to convert Active Record *collection proxy* returned by `user.microposts`
- update User model
  - add `default_scope`
  - pass `order` argument with sql syntax order condition
    - `default_scope -> { order('created_at DESC') }`
  - *DESC* or descending order will return posts in reverse chronological order
- in Rails 4, all scopes take anonymous function which return criteria for scope
  - *lazy evaluation*, scope is evaluated as needed
  - *Proc* (procedure) or *lambda* function
    - takes a block
    - evaluates with the `call` method

#### dependent destory
- currently have the ability to destroy users
- want to destroy posts created by users when destroyed
- to test, we will destory the user and verify the associated posts are no longer in the db
- steps to tests
  - save a local copy of users microposts via `to_a`
  - destroy user
  - make sure local copy is not empty (safety check)
  - iterate over each micropost
    - user `where` to verify they are no in the db and empty obj is returned
    - can also use `find` but we need to check for an *ActiveRecord::RecordNotFound* exception is raised
- updating user model
  - add option to `has_many` associated method
    - `has_many :mircoposts, dependent: :destory`
  - this options specificies that each micro post associated with user should be destroyed with user
- all tests should be green

### 10.1.5 | Content validations
- add validations for tentent
  - must be present
  - can be no longer than 140 char (it's micro afer all)
- update Micropost model spec
  - test for content blank
  - test for content which is too long
    - use string multiplication
- update Micropost model
  - add `validates` for content
    - `validates :content, presence: true, length: { maximum: 140 }`
- all tests passing

## 10.2 | Showing microposts
- no way to create posts through web
- will test display of posts
- ala Twitter, will show posts on user `show` page, not some index page
- will add sample data for now to display

### 10.2.1 | Augmenting the user show page
- use factory to create posts for user, verify contents of user profile page reflect post
- user `let!` to create posts immediately
- test user profile page, in microposts block
  - check for content of both posts
  - check that the count is on the page
- calling count through association
  - does not pull all microposts from db
  - calls count directly on db
  - if cound is still a bottleneck, can use [counter cache](http://railscasts.com/episodes/23-counter-cache-column)
- update the show view
  - user `any?` to check for microposts before rendering dom
  - use microposts count to header of section
  - add ordered list
  - render `@microposts` instance var
    - render each micropost in `@microposts` and use partial
    - looks for a partial, `microposts/_micropost.html.erb`
    - did this for user as well
  - call `will_paginate` on `@microposts` instance var
    - before did not need instance var
    - assumes there is an instance var named after current controlle (`@users`)
    - instance var should be of type ActiveRecord::Relation (sec 9.3.3)
- add a mircoposts partial
  - `app/views/microposts/_micropost.html.erb`
  - render content
  - uses `time_ago_in_words` helper to render human readable created_at
- update the Users controller, show action
  - create a microposts instance var
  - use `paginate` through the `@user` association

### 10.2.2 | Sample microposts
- update `db:populate` rake task to include some microposts
  - update the `lib/tasks/sample_data.rake`
  - for first 6 users (`User.all(limit: 6)`)
    - add 50 microposts
    - use `Faker::Lorem.sentence(5)` to generate content
- update the data base
  - reset database: `bundle exec rake db:reset`
  - populate database: `bundle exec rake db:populate`
  - prepare test db: `bundle exec test:prepare`
- add css to `custom.css.scss` for micropost styling

## 10.3 | Manipulating microposts
- interface for creating/destorying microposts through the web
- will use form to create micropost resource
- first hint of *status feed*
- most Micropost manipulation through Users and StaticPages controllers
  - only need create and destory actions

- microcontroller routes
| HTTP request | URL | Action | Purpose |
-----------------------------------------
| `POST`| /microposts | `create` | create a new micropost |
| `DELETE`| /microposts/1 | `destroy` | destory micropost with id 1 |

### 10.3.1 | Access control
- both `create` and `destory` actions should require user to be signed in
- will add test later to ensure that user only delets their own
- add rspec tests
  - post to "microposts_path" to hit `create`
  - delete to "micropost_path", passing in a micropost Factory Girl to hit `delete`
  - ensure that the response is redirected to sign in page (both cases)
- refactor `signed_in_user` out of the Users controller into the Sessions helper
  - gives access to Users and Microposts controllers
- user a before filter and check for signed in user (`before_action :signed_in_user`)
  - note that if additional actions were added, may have to specify which actions filter applies to
- all tests should pass

### 10.3.2 | Creating Microposts
- following Twitter convention, creating a new post will be on homepage
  - only for signed in user
  - not at seperate path (microposts/new) like user
- serve different homepage based on user sign in status
- generate integration test: `rails generate integration_test micropst_pages`
  - test similar to user page tests
  - subject = page
  - create user with factor girl
  - sign in user before each test
  - describe micropost creation
    - visit root for each test
    - test for invalid info
      - no info
      - check that after click micropost count does not change
      - make sure error messages display
    - test for valid info
      - fill in content field with text
      - check that after click micropost count changes by 1
- update the `create` action for the Mircoposts controller
  - use of strong parameters via `micropost_params` allows only content to be edited through the web
  - create micropost instance var by calling build on user association
  - attempt to save and check result
  - if success
    - flash success
    - redirect to root
  - else if failure
    - render 'static_pages/home'
- update static_pages/home view
  - add check for signed in
    - if signed in render user into and form
    - if not, render original homepage
  - add shared partials
    - user info should have gravatar, link to user and number of posts
    - micropost form should have error messages, text area for content and submit button "Post"
  - update how `error_messages.html.erb` is rendered
    - originally written to anticipate user instance var
    - now needed to be called on user and microposts object
    - the form var `f` can access the object via `f.object` (either `@user` or `@micrpost`)
    - pass `f.object` in as a hash with key *object*
    - update partial to use `object` instead of `@user`
    - update references to rendering this partial, passing in `object: f.object`
      - `users/new`
      - `users/edit`
- update StaticPagesController
  - view expects micropost instance var
  - if user is signed in
    - build one by calling build on `current_user` association
- all specs passing, but not complete

### 10.3.3 | A proto-feed
- will start with basic feed of signed in user's posts on homepage
- ch 11 will introduce a feed of other users posts
- add `feed` method to User model
- add tests for `feed` method
  - verify that user's posts are present
  - verify that other users' posts are not present
  - use `include?` method to check if post is in feed array
- `feed` method implementation
  - instance method
  - get all Micropost where the user_id matches the current users id
  - could use `microposts` associated for same functionality
    - this could resist refactoring when we update the feed
  - `.where(user_id = ?)` the question mark ensures the param passed in is escaped
    - good practice
    - avoid SQL injection
- update static pages test
  - create some microposts
  - for signed in user, make sure that the page has an li + post id on page
- update `home` action in StaticPagesController to add @feed_items
- create partials
  - partial for feed itself
    - seperate partial for feed items
    - will use `collection:` param to pass colleciton of feed items
    - this will call partial on each item in the collection
  - partial for feed item
- update home parital to render feed
- ISSUE: if creating a new micropost fails
  - never redirects to root, does not hit static pages controller
  - partial expects feed instance var
  - init feed instance var to empty array in failing branch of `create` action in mircoposts controller

### 10.3.4 | Destorying microposts
- will add ability to delete a micropost
- add delete links to posts
- delete links will only work for mircoposts created by current user
- update partials
  - check if current user is the same user on the micropost
  - add link with delete method if so
  - update feed item partial and micropost partial
- write tests
  - check that clicking delete link changes the count by 1
- update microposts controller
  - add correct user method to verify that current user is micropost author
  - add before filter to destory action with correct user method
  - can use `find_by` and check for nil or use `find` and catch exception


## 10.4 Conclusion
- done with micropost resource
- need to add social layer in next section
- commit to master
- push to heroku


## 10.5 Exercises
1. added test to static page test and checked result of pluralize
2. create 40 add'l microposts and add check that number of posts (li within container) is equal to pagination default==30
3. (done)
4. on user page spec, check for no delete link when not signed in, link when signed in on own page, and no link on another page when signed in
5. (done)
6. add helper method to micrposts helper and call on content
7. add jquery handler to watch keyup from element, add element to track feedback




