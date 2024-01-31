---
url: https://dev.to/miry/unlocking-performance-installing-ruby-with-yjit-on-macos-3iei
canonical_url: https://dev.to/miry/unlocking-performance-installing-ruby-with-yjit-on-macos-3iei
title: 'Unlocking Performance: Installing Ruby with YJIT on MacOS'
slug: unlocking-performance-installing-ruby-with-yjit-on-macos-3iei
description: 'Boosting Ruby Performance on MacOS: A Guide to Installing YJIT.'
tags:
- ruby
- yjit
- macos
- howto
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2024-01-04-unlocking-performance-installing-ruby-with-yjit-on-macos-3iei-cover_image-https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F94rwktpgvji3k4jwupy0.png)

# Unlocking Performance: Installing Ruby with YJIT on MacOS


> Foto von <a href="https://unsplash.com/de/@joshuafuller?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Joshua Fuller</a> auf <a href="https://unsplash.com/de/fotos/roter-edelstein-XzuJuyYLjmE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

YJIT, the groundbreaking Just-In-Time compiler for Ruby, brings a significant performance boost to your applications. However, it's important to note that YJIT isn't enabled by default in ruby. Fear not, though – with a few straightforward steps, you can harness the power of YJIT on your MacOS system.

To begin, it's essential to rebuild Ruby with the proper configuration option. This involves using the `--enable-yjit` flag [^1]. Additionally, make sure you have `rustc` installed, as YJIT relies on it.

For users leveraging `ruby-build`, incorporating compile options is a breeze through environment variables [^2]. This allows for a seamless integration of YJIT into your Ruby environment, ensuring that your applications run with enhanced speed and efficiency.

```shell
$ brew install rust
$ RUBY_CONFIGURE_OPTS="--enable-yjit" rbenv install 3.3.0

$ rbenv shell 3.3.0

$ ruby --yjit -v
ruby 3.3.0 (2023-12-25 revision 5124f9ac75) +YJIT [arm64-darwin23]

$ ruby --yjit -e "p RubyVM::YJIT.enabled?"
true

$ ruby -e "RubyVM::YJIT.enable; p RubyVM::YJIT.enabled?"
true
```

These shell instructions provide a step-by-step guide to installing additionaly Rust, configuring YJIT, installing Ruby 3.3.0 with YJIT support, and checking if YJIT is enabled both from the command line and within a Ruby script. Once `rustc` is installed, there's no longer a need to specify the `RUBY_CONFIGURE_OPTS` environment variable, as ruby will be built with YJIT enabled. However, it is included for explicitness, just in case there are any missing dependencies.

Excitingly, with the release of Ruby 3.3.0, there are new possibilities for enabling YJIT directly from your code (run-time) [^3]. This empowers developers with greater flexibility and control, enabling them to fine-tune their Ruby setup according to their specific needs.

```ruby
RubyVM::YJIT.enable if defined? RubyVM::YJIT.enable
```

In conclusion, elevating your Ruby experience with YJIT is a straightforward process that reaps substantial benefits. By following these steps and staying abreast of the latest Ruby updates, you can unlock a new level of performance for your applications.

## References

[^1]: [YJIT Installation instructions](https://github.com/ruby/ruby/blob/master/doc/yjit/yjit.md#installation)
[^2]: [ruby-build README](https://github.com/rbenv/ruby-build#readme)
[^3]: [Ruby 3.3.0 Released](https://www.ruby-lang.org/en/news/2023/12/25/ruby-3-3-0-released)


