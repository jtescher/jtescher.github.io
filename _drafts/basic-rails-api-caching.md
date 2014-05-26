---
layout: post
title:  "Basic Rails API Caching"
date:   2014-05-26 09:00:00
categories: rails api caching
---

Rails performance out of the box is acceptable for prototypes and small applications with limited traffic. However as 
your application grows in popularity you will inevitably be faced with the decision to either add more servers, or use
your existing servers more efficiently. Complex caching strategies can be incredibly difficult to implement correctly,
but simple caching layers can go a long way.

In this post I'll explain two basic rails caching mechanisms and explain some of the costs and benefits of each.


HTTP Caching
------------

If your API responses are mostly static content like a list of available products, then 
[HTTP Caching](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html) can be a very effective solution. Even something
as low as a one minute cache can move the vast majority of your requests to a CDN or an in-memory store like 
[rack cache](http://rtomayko.github.io/rack-cache/).

Specifying the expiration time is simple. In your controller action just call `expires_in` with the time:

```ruby
class ProductsController < ApplicationController
  def index
    expires_in 1.minute, public: true
    @products = Product.all
  end
end
```

This will result in a header being set to `Cache-Control: max-age=60, public` which any CDN will pick up and serve for
you instead of the request hitting your server.

This solution works well when the content is mostly static but it comes with the downside that changes to your content
will not be seen for up to one minute (or whichever time you have chosen).


Conditional GET
---------------

Another option is using Etags or Last-Modified times to know what version of the resource the client has last seen and 
returning a HTTP `304 Not Modified` response with no content if the resource has not changed.

To set this up in a controller you can either use the
[fresh_when](http://api.rubyonrails.org/classes/ActionController/ConditionalGet.html#method-i-fresh_when) or 
[stale?](http://api.rubyonrails.org/classes/ActionController/ConditionalGet.html#method-i-stale-3F) methods. Here is an 
example using the `fresh_when` method.

```ruby
class ProductsController < ApplicationController
  def show
    @product = Product.find(params[:id])
    fresh_when(etag: @product, last_modified: @product.created_at, public: true)
  end
end
```

This method attaches an `ETag` header and a `Last-Modified` header to every product response. Now if you make a request 
for a given product you will see the headers in the response:

```bash
curl -i localhost:3000/products/1
HTTP/1.1 200 OK
ETag: "91206795ac4c5cd1b02d8fcbc752b97a"
Last-Modified: Mon, 27 May 2014 09:00:00 GMT
...
```

And if you make the same request but include the etag in a `If-None-Match` header, the server can return 304 with empty
content and save all the time it would have spent rendering the content.
 
```bash
curl -i localhost:3000/products/1.json \
  --header 'If-None-Match: "91206795ac4c5cd1b02d8fcbc752b97a"'
HTTP/1.1 304 Not Modified
Etag: "91206795ac4c5cd1b02d8fcbc752b97a"
Last-Modified: Mon, 27 May 2014 09:00:00 GMT
...
```

The other option is to use the `If-Modified-Since` header in the request, which will have the same result:

```bash
curl -i localhost:3000/products/1.json \
  --header 'If-Modified-Since: Mon, 27 May 2014 09:00:00 GMT'
HTTP/1.1 304 Not Modified
Etag: "91206795ac4c5cd1b02d8fcbc752b97a"
Last-Modified: Mon, 27 May 2014 09:00:00 GMT
...
```

This method still requites a request to be made to the Rails app, and the product still has to be pulled from the 
database to determine the `created_at` time. However rendering the response can be a substantial portion of
each request so this is a simple way to save a lot of time.

These options are only the beginning of the caching options rails offers. As of rails 4 
[page caching](https://github.com/rails/actionpack-page_caching) as well as 
[action caching](https://github.com/rails/actionpack-action_caching) have been pulled out into their own gems and are 
worth looking at if you need those options.

Finally if you are ready for something a little more robust you can read about Basecamp's 
[Russian Doll Caching](http://signalvnoise.com/posts/3113-how-key-based-cache-expiration-works) to see how they solve
these problems.
