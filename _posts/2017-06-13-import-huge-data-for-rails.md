---
url: https://medium.com/notes-and-tips-in-full-stack-development/import-huge-data-for-rails-e7e1b2713d71
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/import-huge-data-for-rails-e7e1b2713d71
title: Import Huge Data for Rails
subtitle: 'I wrote a simple rake task for dump data from DB to CSV file:'
slug: import-huge-data-for-rails
description: ""
tags:
- ruby-on-rails
author: Michael Nikitochkin
username: miry
---

# Import Huge Data for Rails

![Image: Sundance 2016 Brief Sizzle: The Leviathan Project](/assets/2017-06-13-import-huge-data-for-rails-1_lMrrSmkgBB3IvwuZz4azIg.jpeg)

I wrote a simple rake task for dump data from DB to CSV file:

```
namespace :db do
  desc 'Create CSV fixtures from data'
  task :extract_to_csv => :environment do
    ActiveRecord::Base.establish_connection
    skip_tables = ["schema_info", "schema_migrations"]

    (ActiveRecord::Base.connection.tables - skip_tables).each do |table_name|
      FasterCSV.open("#{RAILS_ROOT}/db/fixtures/#{table_name}.csv", "w", :force_quotes => true) do |csv|
        model = table_name.classify.constantize
        csv << model.column_names
        model.all.each do |object|
          csv << model.column_names.map{|c| object.attributes[c]}
        end
      end
    end
  end
end
```
> *[import-task.rb view raw](https://gist.githubusercontent.com/marchi-martius/1ed14cef4ecd21f369608a80605d52c1/raw/85a187fd68415e55fdf308381785f5ece8bf9c9b/import-task.rb)*

And import for Postgresql:

```
copy import_products from '/home/miry/import_products.csv'
  with csv header NULL AS '' QUOTE  AS  '"';
```
> *[import-script.sql view raw](https://gist.githubusercontent.com/marchi-martius/afc3b6bf75eb1b1d7f277e64e274ef24/raw/af11b9a3cc304374abc27fc0616325e27260d6a0/import-script.sql)*


