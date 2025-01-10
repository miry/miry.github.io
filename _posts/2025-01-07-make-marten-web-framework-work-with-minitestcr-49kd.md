---
url: https://dev.to/miry/make-marten-web-framework-work-with-minitestcr-49kd
canonical_url: https://dev.to/miry/make-marten-web-framework-work-with-minitestcr-49kd
title: Make Marten Web Framework work with minitest.cr
slug: make-marten-web-framework-work-with-minitestcr-49kd
description: This article demonstrates how to integrate the Marten Web Framework with
  minitest.cr for testing.
tags:
- crystal
- programming
- marten
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2025-01-07-make-marten-web-framework-work-with-minitestcr-49kd-cover_image-zv17ebb2iutt49vza767.jpeg)

# Make Marten Web Framework work with minitest.cr


## Summary

This article demonstrates how to integrate the Marten Web Framework with minitest.cr [^1] for testing. For initializing a new project, refer to the Marten documentation on installation [^2].

## Instructions

### Step 0: Initialize a Simple Task Manager with Rakefile

It is an optional step. 
Create a `Rakefile` with the following content:

```ruby
# Rakefile
task test: ["test:all"]
namespace :test do
  desc "Run tests"
  task :all, [:path] do |t, args|
    args.with_defaults(path: pwd)
    option_list = ENV.fetch("TESTOPTS", "--verbose")
    option_list = "-- #{option_list}" if option_list.size > 0

    files = FileList["#{args.path}/**/*_test.cr"].reject { |f| f.include?("/lib/") }.join(" ")
    sh "crystal run --error-trace #{files} #{option_list}" if files.size > 0
  end
end
```

This setup allows you to run tests from any folder with customizable options. Here are some usage examples:

```sh
$ rake test
$ TESTOPTS="-v -n /response/" rake test
$ TESTOPTS="--help" rake test
$ cd test/handler; rake test
```

### Step 1: Install `minitest.cr`

Add the following lines to your `shard.yml` file:
```yml
# shard.yml
development_dependencies:
  minitest:
    github: ysbaddaden/minitest.cr
```

Then, run:

```sh
$ shards install
```

### Step 2: Set Up the Project in Tests

Create an initial `test/test_helper.cr` with the following content, based on the Marten testing basics documentation [^3]:

```crystal
ENV["MARTEN_ENV"] = "test"

require "spec"
require "marten"
require "marten/spec"

require "../src/project"
```


I slightly modified it made it work with minitest
Update the `test_helper.cr` as follows:

```crystal
# test/test_helper.cr
ENV["MARTEN_ENV"] = "test"

require "minitest/autorun"
require "marten"
require "marten/cli"

# Only require those to avoid running Crystal Spec in parallel with minitest
require "marten/spec/ext/db/statement"
require "marten/spec/client"

# Copy from https://github.com/martenframework/marten/blob/main/src/marten/spec.cr
module Marten
  module Spec
    @@client : Marten::Spec::Client?

    def self.clear_client : Nil
      @@client = nil
    end

    def self.clear_collected_emails : Nil
      Marten::Emailing::Backend::Development.delivered_emails.clear
    end

    def self.delivered_emails : Array(Emailing::Email)
      Marten::Emailing::Backend::Development.delivered_emails
    end

    def self.client
      @@client ||= Marten::Spec::Client.new
    end

    def self.flush_databases
      Marten::DB::Connection.registry.values.each do |conn|
        Marten::DB::Management::SchemaEditor.run_for(conn) do |schema_editor|
          schema_editor.flush_model_tables
        end
      end
    end

    def self.setup_databases
      Marten::DB::Connection.registry.values.each do |conn|
        if !conn.test_database?
          raise "No test database name is explicitly defined for database connection '#{conn.alias}', cancelling..."
        end

        Marten::DB::Management::SchemaEditor.run_for(conn) do |schema_editor|
          schema_editor.sync_models
        end
      end
    end
  end
end

# Hooks defined in https://github.com/ysbaddaden/minitest.cr/blob/bfc73a6196129e59b6c05d49d69e542f83a5939f/src/test.cr#L24
class Minitest::Test
  def before_setup
    # Register applications only once
    Marten.setup if Marten.apps.app_configs.empty?
    Marten::Spec.setup_databases
  end

  def after_teardown
    Marten::Spec.flush_databases
    Marten::Spec.clear_collected_emails
    Marten::Spec.clear_client
    DB::Statement.reset_query_count
  end
end

require "../src/project"
```


**Explanation:**
* **Replacing `require "marten/spec"`**: Instead of using the default `marten/spec`, this setup avoids running crystal spec alongside `minitest`.
* **Adding Hooks**: Minitest hooks (`before_setup` and `after_teardown`) handle database setup and cleanup.
* **Additional Requirements**: Extensions for simple tests:
  ```crystal
  require "marten/spec/ext/db/statement"
  require "marten/spec/client"
  ```


### Step 3: Test Model

Here is a sample model test file `test/models/page_description_test.cr`:

```crystal
# test/models/page_description_test.cr
require "../test_helper.cr"

class PageDescriptionTest < Minitest::Test
  def test_create_element
    obj = PageDescription.new(name: "foo")
    obj.save
    assert_equal "foo", obj.name
  end
end
```

### Step 4: Test Handler

Here is a handler test file `test/handlers/api/page_descriptions_handler_test.cr`:

```crystal
# test/handlers/api/page_descriptions_handler_test.cr
require "../../test_helper.cr"

class API::PageDescriptionsHandlerTest < Minitest::Test
  def client
    @client ||= Marten::Spec.client
  end

  def test_response_status
    PageDescription.create(name: "print", lang: "uk", template_key: "top")
    PageDescription.create(name: "print", lang: "uk", template_key: "bottom")

    response = client.get("/api/page_descriptions/uk", query_params: {"name" => "print"})

    assert_equal 200, response.status
    assert_equal "application/json", response.content_type

    body = JSON.parse(response.content)
    assert_equal "<body placeholder>", body.dig("top", "body")
    assert_equal "<new_pressman_body placeholder>", body.dig("top", "new_pressman_body")
    assert_equal "<body placeholder>", body.dig("bottom", "body")
  end
end
```

### Result

Run all test and would produce the output

```sh
$ rake test
crystal run --error-trace test/models/page_description_test.cr test/requests/api/page_descriptions_test.cr -- --verbose
Run options: --seed 33746 --verbose --parallel 1

API::PageDescriptionsTest#test_response_status = 0.110 s = .
PageDescriptionTest#test_create_element = 0.011 s = .

Finished in 00:00:00.120904917, 8.270962214051229 runs/s

2 tests, 0 failures, 0 errors, 0 skips
```

---

![That's all Folks](/assets/2025-01-07-make-marten-web-framework-work-with-minitestcr-49kd-w79xr8tiu3s02n5ta1m3.png)

### To be continued...

Once I explore additional cases, I will update these examples.

## References

[^1]: [minitest.cr: Test Unit for the Crystal programming language](https://github.com/ysbaddaden/minitest.cr)
[^2]: [Marten Docs: Installation](https://martenframework.com/docs/getting-started/installation)
[^3]: [Marten Docs: Testing](https://martenframework.com/docs/development/testing#the-basics)


