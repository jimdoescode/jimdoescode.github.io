---
layout: "post"
author: "jimdoescode"
title: "Handle dynamic file uploads when testing a rails controller method"
date: "2021-06-23 13:50:14"
tags: [ruby,rails,rspec,testing]
---

The premise is this: You're testing a rails controller method that takes a CSV file upload. How do you make a CSV (or whatever file you want) that contains IDs that are unique on each test run? 

Seems like a pretty straightforward problem and yet the answers from every single search I made online just used some static CSV (or whatever file) stashed in a directory accessible to the test. This doesn't work for me. The IDs in my test can change on each run as the test data gets hydrated. Also what if that static file gets deleted or moved? Now your test breaks when none of the logic it was testing changed.

Here's the solution I came up with to solve this in my controller spec test. Again, I'm using a CSV here but you can do this with any kind of file you want.

```ruby
person = Person.create(dob: '01/01/1970', name: 'Test Guy') # This creates a new record with some kind of ID value
# Do some validation on the record and whatever other things you want to do 

csv = Tempfile.new('test.csv')
# Write the header
csv.write("ID,DOB,Name\n")
@ Write the data 
csv.write("#{person.id},#{person.dob},#{person.name}\n"
csv.rewind

controllerParams = {
    file: Rack::Test::UploadedFile.new(csv)
    ... # Whatever other params get POSTed to your controller
}

post :csv_import params: contorllerParams
```

So stepping through the above code. We make our record using whatever method. There is no "create" method. I'm just making that up for the demonstration. If you need to do something after you've made or fetched the record then do that. 

Finally when you're ready to build your CSV use the [Tempfile](https://ruby-doc.org/stdlib-3.0.1/libdoc/tempfile/rdoc/Tempfile.html) class and populate your file data. Once you've got that all set just call `Rack::Test::UploadFile.new` and pass in the temp file then you just call your controller method with the parameters you need.

Happy Testing 👍 
