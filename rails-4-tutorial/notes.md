# Ruby On Rails Tutorial
## *by Michael Hartl* ##

*This book provides a tutorial covering Ruby basics, Rails basics, deployment with Heroku, and building a twitter clone from the ground up. This book has multiple versions, these notes are for the Rails 4 version*



# working chapter #############################################################################


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


## 7.2 | Sign up form



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





# Chapter 8 | Sign in, Sign out #############################################################################

#### Topics:

#### References:



# Chapter 9 | title #############################################################################

#### Topics:

#### References:



# Chapter 10 | title #############################################################################

#### Topics:

#### References:



# Chapter 11 | title #############################################################################

#### Topics:

#### References:




