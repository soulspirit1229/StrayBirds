# Rails consile中的object

## app
~~~rb
>> app.project_path(Project.first)
=> "/projects/130349783-with-attachments"

>> app.get "/735644780/projects/605816632-bcx.atom" 
=> 200

>> app.response.body
=> "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<feed xml:lang=\"en-US\"
~~~

## controller
## helper
## new_session
## reload!