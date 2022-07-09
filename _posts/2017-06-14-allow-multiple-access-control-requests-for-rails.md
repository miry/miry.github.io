---
url: https://medium.com/notes-and-tips-in-full-stack-development/allow-multiple-access-control-requests-for-rails-50d1162c1425
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/allow-multiple-access-control-requests-for-rails-50d1162c1425
title: Allow Multiple Access Control Requests for Rails
subtitle: If you have problem with sending XHR requests from different domains, it
  will be problematic to get content, because
slug: allow-multiple-access-control-requests-for-rails
description: ""
tags:
- cors
author: Michael Nikitochkin
username: miry
---

# Allow Multiple Access Control Requests for Rails

If you have problem with sending XHR requests from different domains, it will be problematic to get content, because

```
XMLHttpRequest cannot load http://different.domain.local:3000/visits. Origin http://localhost:3000 is not allowed by Access-Control-Allow-Origin.
```

The solution is to setup response headers: *Access-Control-Request-Method*, *Access-Control-Allow-Origin*.

```
headers['Access-Control-Allow-Origin'] = '*'
headers['Access-Control-Request-Method'] = '*'
```
> *[cors-headers.rb view raw](https://gist.githubusercontent.com/marchi-martius/603299d49b5eb1d38e8fdf0bded34ce8/raw/bc34757dbe7509abae7f1d599db90344012cace2/cors-headers.rb)*

Example for rails:

```
#app/application_controller.rb
after_action :set_access_control_headers
def set_access_control_headers
  headers['Access-Control-Allow-Origin'] = '*'
  headers['Access-Control-Request-Method'] = '*'
end
```
> *[cors-rails.rb view raw](https://gist.githubusercontent.com/marchi-martius/b08591437964571955b4b198b00d72c8/raw/cdf8dca487496d952e2197565ee5667a293923bb/cors-rails.rb)*

You can look into sources of the sample Sinatra application in [GitHub](https://github.com/miry/cross-domain-sharing). Demo live in [Heroku](http://cross-domain-ajax-request.herokuapp.com/index.html). Or use [these samples](https://gist.github.com/miry/5447203).


