OctoDB on Ruby
==============

We can interface with OctoDB using the [sqlite3-ruby](https://github.com/sparklemotion/sqlite3-ruby) wrapper


### Installation

First follow the instructions to compile and install OctoDB from the source code or use
the pre-compiled binaries for your platform. You can start with the free version

Then you can install the wrapper according to the platform:

#### On Linux and Mac

    sudo gem install sqlite3 -- --with-sqlite3-include=/usr/local/include \
          --with-sqlite3-lib=/usr/local/lib/octodb/

#### On Windows

    gem install sqlite3


### Example Code

```ruby
require "sqlite3"
require "json"

# Open a database
uri = "file:test.db?node=secondary&connect=tcp://127.0.0.1:1234"
db = SQLite3::Database.new uri

# check if the db is ready
loop do
  result = db.get_first_value "PRAGMA sync_status"
  status = JSON.parse(result)
  break if status["db_is_ready"] == true
  sleep 0.5
end

# now we can access the db
p "the database is ready"

db.execute("INSERT INTO users (name, email, phone) VALUES (?, ?, ?)",
           ["John", "john@email.com", ""])

db.execute("select * from users") do |row|
  p row
end
```
