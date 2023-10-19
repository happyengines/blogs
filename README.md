# code block
## code block test
code block:

```ruby
def setup_database
  puts "Database connection details:#{params['database'].inspect}"
  return unless params['database']
  # estabilsh database connection
  ActiveRecord::Base.establish_connection(params['database'])
end
```

image：

![パンダ](images/panda.png  "画像タイトル")

bb
