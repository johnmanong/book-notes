Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

Chapter N | title #############################################################################

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

# Chapter 2 #############################################################################

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


# Chapter 1 #############################################################################
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





HEROKU ##################################################################################

$ heroku create						// create heroku side repo
$ git push heroku master			// push to heroku
$ heroku rename railstutorial		// rename repo on heroku (optional, must be unique)
$ heroku logs						// view logs