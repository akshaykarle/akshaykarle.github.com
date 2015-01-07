---
layout: article
title:  "Gotchas with Rails caching and Nginx"
date:   2014-09-17 22:00:00
categories: tech
tags: [rails, caching, nginx, etags]
comments: true
---

Rails allows you to [cache fragments of the view](http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching) thus saving time by avoiding performing database operations and rendering the views from the template. The cache generates an ETag which is sent by the server to the client. It then uses the [conditional GETs](http://guides.rubyonrails.org/caching_with_rails.html#conditional-get-support) from the HTTP specification to tell the clients whether they need to update the view or not. Conditional GET uses two headers `HTTP_IF_NONE_MATCH` and `HTTP_IF_MODIFIED_SINCE` as part of the request which contains an ETag and a timestamp of last updated at time of the view.

Sample HTTP response from Rails containing the ETag:
![Sample HTTP response from Rails containing the ETag](/assets/2014-09-17-rails-caching/etag-response-header.png)

Sample HTTP request from the client containing the HTTP_IF_NONE_MATCH:
![Sample HTTP request from the client containing the HTTP_IF_NONE_MATCH](/assets/2014-09-17-rails-caching/etag-request-header.png)

The Rails server looks at modified time and the ETag supplied by the clients to determine whether to re-generate the view or to reuse the cache. ETag validations are of two types - [Strong and Weak](http://en.wikipedia.org/wiki/HTTP_ETag#Strong_and_weak_validation). A Strong ETag match indicates that the two resource match byte by byte where as a Weak ETag indicates that the resources are semantically equivalent. A Weak Etag is same as a Strong ETag except that it is prefixed by a "W/". Why am I talking about Strong and Weak ETags you might wonder? You will come to answer in just a minute.

Nginx 1.7.3 added support for preserving ETags on response modifications as you can see in the [changelog](http://nginx.org/en/CHANGES). A form of response modification could be [gzipping the response](http://nginx.org/en/docs/http/ngx_http_gzip_module.html) which is commonly used to reduce the size of transmitted data to the client. When the response is gzipped the content is modified and it is not comparable byte by byte with the content of the server. Hence, Nginx actually replaces the Strong ETag sent by rails with a Weak ETag.

For eg - If the ETag that rails generated was "123456789", Nginx would update the ETag to W/"123456789".

Sample HTTP response from Nginx containing the Weak ETag:
![Sample HTTP response from Nginx containing the Weak ETag](/assets/2014-09-17-rails-caching/weak-etag-response-header.png)

Sample HTTP request from the client containing the HTTP_IF_NONE_MATCH:
![Sample HTTP request from the client containing the HTTP_IF_NONE_MATCH](/assets/2014-09-17-rails-caching/weak-etag-request-header.png)

The client looks at the Weak ETag sent by Nginx and sends it back to rails in the `HTTP_IF_NONE_MATCH` header. When rails receives this ETag prefixed with W/ it doesn't match with the ETag it had originally computed since Rails is not aware about the gzipping that Nginx did. This results in a cache miss and Rails regenerates the content again. This could result in high response times since you are never actually hitting the cache.

The fix for was simply to ignore the starting "W/" for the Weak ETags before processing the request.

{% highlight ruby %}
class WeakEtagMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    etag = env['HTTP_IF_NONE_MATCH']

    if etag && etag.match(/^W\//)
      env['HTTP_IF_NONE_MATCH'] = etag.gsub(/^W\//, '')
    end

    status, headers, body = @app.call(env)
    [status, headers, body]
  end
end
{% endhighlight %}

Adding a middleware which does this seems to work. Now when rails receives the ETag, it doesn't receive a Weak ETag and the ETags match perfectly and the caching works as expected.
