---
url: https://dev.to/miry/recording-my-crystal-snippets-from-todays-learning-21ej
canonical_url: https://dev.to/miry/recording-my-crystal-snippets-from-todays-learning-21ej
title: Recording My Crystal Snippets from Today’s Learning
slug: recording-my-crystal-snippets-from-todays-learning-21ej
description:
  "I want to document some snippets from today’s learning while working
  on open-source projects. "
tags:
  - crystal
  - programming
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2025-03-15-recording-my-crystal-snippets-from-todays-learning-21ej-cover_image-pzxesom48ukrnql79f11.jpeg)

# Recording My Crystal Snippets from Today’s Learning

I want to document some snippets from today’s learning while working on open-source projects.

## 0. Printing Available Methods for an Object of a Class

I found it useful to debug an object’s methods in a way similar to Ruby. Here’s a snippet that helped me with this:

<!-- {% raw %} -->

```crystal
# Print the available methods of a Crystal class
# Usage: `puts Spec::CLI.methods.sort`
class Object
  macro methods
    {{ @type.methods.map &.name.stringify }}
  end
end

puts Spec::CLI.methods.sort # => ["abort!", "add_formatter", ...]
```

<!-- {% endraw %} -->

More macro examples can be found in the [Crystal Macro Methods](https://crystal-lang.org/reference/1.15/syntax_and_semantics/macros/macro_methods.html) documentation.

![phantasy crystal spec](/assets/2025-03-15-recording-my-crystal-snippets-from-todays-learning-21ej-jgckammc4harxop12z3h.jpeg)

## 1. Filtering Crystal Spec Tests Based on Tags

I worked on executing different types of tests, including unit and integration tests.  
There are multiple approaches to separating them, and many ideas can be found in this [Crystal Forum discussion](https://forum.crystal-lang.org/t/exclude-all-tests-with-tags/6861/1).

Here’s the approach I took:

### Project Folder Structure

My project’s test structure looks like this:

```
$ tree spec
spec
├── awscr-s3
├── awscr-s3_spec.cr
├── fixtures.cr
├── integration
│   ├── compose.yml
│   └── minio_spec.cr
└── spec_helper.cr
```

The integration tests are marked with the tag `"integration"`.

### Filtering Tests Based on Tags

One of the things I love about Crystal is that the code is simple and intuitive to read.  
While learning about the `Spec` module in the [Crystal Spec Documentation](https://crystal-lang.org/api/master/Spec.html), I found links to the source code, which helped me understand how filtering works.

Here’s how I implemented tag-based filtering:

```crystal
# spec/spec_helper.cr
class Spec::CLI
  def tags
    @tags
  end
end

Spec.around_each do |example|
  tags = Spec.cli.tags
  # By default, skip tagged tests and run only unit tests
  next if (tags.nil? || tags.empty?) && !example.example.all_tags.empty?
  example.run
end
```

#### Explanation

`Spec.cli` is a command-line interface that parses options and stores them internally in the `@tags` variable.

For example, when running:

```
$ crystal spec --tag 'integration'
```

The `"integration"` tag is stored as a `Set` in `@tags`. This allows me to check which filters were enabled without manually parsing the command-line arguments.

However, there’s a small drawback: `@tags` is not publicly accessible. To work around this, I extended the `Spec::CLI` class and exposed it. (There may be a better way to do this.)

The second part of the code is a simple filtering mechanism implemented using `Spec.around_each`:

- It checks the provided tags and then validates the test’s tags.
- If no tags are specified, all tagged tests are skipped by default.

A simple debug statement like `pp! example` can help explore more filtering options.

## 2. Configuring Test Dependencies Based on Tags

Integration tests allow sending real requests.  
Instead of adding tags to every integration test individually, we can leverage the folder structure (e.g., placing them in an `integration` folder).

Here’s one way to configure `WebMock` dynamically based on test tags or file location:

```crystal
Spec.around_each do |example|
  integration = example.example.all_tags.includes?("integration") || example.example.file.includes?("spec/integration")
  WebMock.reset

  WebMock.allow_net_connect = integration
  example.run
end
```

---

That’s all for today! 🚀

![That's all folks](/assets/2025-03-15-recording-my-crystal-snippets-from-todays-learning-21ej-4iav7tzz2q1zwal1r5i1.png)
