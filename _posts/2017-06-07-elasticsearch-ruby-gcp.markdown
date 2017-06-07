---
layout: post
comments: true
title:  "Deploying Rails with Searchkick to GCP"
date:   2017-06-07 02:31:51 -0400
categories: blog
---

# Building a Rails app with Elasticsearch using the searchkick gem and deploying to Google Cloud Platform


In this post I will explain how I created a simple Rails prototype to test searchkick with ElasticSearch, and deployed to App Engine and Cloud SQL. Also how to deploy a ElasticSearch cluster in Compute Engine and make them both work together in production. My goal was to test the Autocomplete feature.

You can find my code [here](https://github.com/gumantov/greatbuy)

First thing you need to do is create the app by running the good and old rails new app command, and then cd to your directory. Secondly, you need to add searchkick in your Gemfile and bundle install.

After that, what I did in order to have a good understanding how ElasticSearch works, I went ahead and downloaded a decent size data set freely available on GitHub in order to test my queries. You can find the dataset that I used right [here](https://github.com/BestBuyAPIs/open-data-set). To make things more interesting and see how it would work using Google Cloud Platform, I also created a CLoud SQL instance at Google Cloud and inserted the same data set to Cloud SQL. All my search queries would be handled by ElasticSearch and normal queries would be handled by CloudSQL.

In order to seed my database, I created this simple script at the db/seeds.rb file. Basically it was simply parsing the json objects and making ActiveRecord create and write to my database.

```ruby
require 'json'

file = File.read('/Users/open-data-set/products.json')
product_hash = JSON.parse(file)

product_hash.each do |i|
  pro = Product.new
  pro.name = i['name']
  pro.sku = i['sku']
  pro.kind = i['type']
  pro.price = i['price']
  pro.category = i['category']
  pro.shipping = i['shipping']
  pro.description = i['description']
  pro.manufacturer = i['manufacturer']
  pro.model = i['model']
  pro.url = i['url']
  pro.image = i['image']
  pro.save
end
```     
### Setting up searchkick

Searchkick is a great gem in Ruby to create a search engine using ElasticSearch in minutes. All the documentation about searchkick you can find it [here](https://github.com/ankane/searchkick).

Here is the code that I used to make it work.

In my model, *product.rb*

```ruby
class Product < ApplicationRecord
  searchkick word_start: [:name, :sku]
end
```

In the controller you need to create a action for your search. I created 2 actions, one for autocomplete and one for search, because I want the search results in a different page. If you want in your main page, you can create this action in your index.

*products_controller.rb*
```ruby
def search
  if params[:search].present?
    @products = Product.search(params[:search], load: false, page: params[:page], per_page: 20)
  else
    @products = Product.paginate(:page => params[:page], :per_page => 20)
  end
end

def autocomplete
  render json: Product.search(params[:query], {
    fields: ["name^5", "sku"],
    match: :word_start,
    limit: 10,
    load: false,
    misspellings: {below: 5}
  }).map do |product|
    { title: product.title, value: product.id }
  end

```

This is the view I created in order to have it all working.

*_header.html.erb*

```
<!-- app/views/layouts/_header.html.erb -->
<%= form_tag search_products_path, method: :get, class: "navbar-form navbar-right", role: "search" do %>
  <p>
    <%= text_field_tag :search, params[:search], class: "form-control typeahead", id: "products_search", autocomplete: :off, placeholder: 'Search' %>
    <%= submit_tag "Search", name: nil, class: "btn btn-default" %>
  </p>
<% end %>

```

Finally, in order to have the autocomplete feature working, we need to do some JavaScript coding to make sure, we are receiving the key strokes and querying the Elasticsearch cluster.

I used Typehead library in order to help me with this piece. It took me some time but this is the final code that worked. Remember to install jquery and Typehead in your JavaScript assets.

*assets/application.js*
```
//= require jquery
//= require jquery_ujs
//= require rails-ujs
//= require turbolinks
//= require_tree .
//= require typeahead.bundle
```
*javascripts/autocomplete.js*

```javascript
var ready;
ready = function() {
    var numbers = new Bloodhound({
      remote: {
        url: "/products/autocomplete?query=%QUERY",
        wildcard: '%QUERY'},
      datumTokenizer: function(d) {
              return Bloodhound.tokenizers.whitespace(d.name); },
      queryTokenizer: Bloodhound.tokenizers.whitespace


});

// initialize the bloodhound suggestion engine

var promise = numbers.initialize();

promise
.done(function() { console.log('success!'); })
.fail(function() { console.log('err!'); });

// instantiate the typeahead UI
$( '.typeahead').typeahead(null, {
  displayKey: 'name',
  source: numbers.ttAdapter()
});
}

$(document).ready(ready);
$(document).on('page:load', ready);

```

### Deploying the Rails app to App Engine and Cloud SQL (Postgres Beta)

I deployed my app to App Engine and used the GCP Cloud SQL Postgresql which was just released and it is in beta right now. It works very easily, and I found much friendlier to work than MySQL.

For the elasticsearch, I created one Ubuntu instance in GCE and installed elasticsearch with apt-get, that's pretty much all. Searchkick by defautl will come with ENV["ELASTICSEARCH_URL"] default to http://localhost:9200 configured to query elasticsearch. You need to change this ENV to the ip address of your instance. This was just for testing, so in production you would need to take a more secure approach.

I decided to install the CLoudSQL proxy in my laptop in order to do all the testing and seed the Database. You can find all the information at the Google Cloud webpage [here](https://cloud.google.com/sql/docs/postgres/connect-admin-proxy). This **[CodeLab](https://codelabs.developers.google.com/codelabs/cloud-ruby-on-rails-cloud-sql-postgres-ruby/index.html?index=..%2F..%2Findex#0)** will also help you a lot.

*config/database.yml*
```
development:
  adapter: postgresql
  database: compramedb_dev
  encoding: utf8
  host: "/cloudsql/[my project]"
  password: [my password]
  pool: 5
  timeout: 5000
  username: user


production:
  adapter: postgresql
  database: compramedb
  encoding: utf8
  host: "/cloudsql/[my project]"
  password: [my password]
  pool: 5
  timeout: 5000
  username: user


test:
  adapter: postgresql
  database: greatbuy_test
  encoding: unicode
  password: [my password]
  pool: 5
  username: user
```

The last piece of the puzzle was configure the app.yml file to push to App Engine.

*app.yml*
```
entrypoint: bundle exec rackup --port $PORT
env: flex
runtime: ruby

env_variables:
  SECRET_KEY_BASE: your secret key
  ELASTICSEARCH_URL: http://publicIPofGCEinstance:9200
  GREATBUY_DATABASE_PASSWORD: [your password]


beta_settings:
    cloud_sql_instances: [cloudsql connection name]
```

`gcloud app deploy` and you should see your app in the cloud. It's magical when it works.

Here is the app, look at the search bar and the autocomplete working. It is suggesting by product name.

![comprameapp](https://storage.googleapis.com/cloudnativeblog-images/Images/comprame.png)
