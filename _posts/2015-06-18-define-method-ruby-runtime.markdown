---
layout: post
title:  "Using define_method to cleanup your code"
date:   2015-06-18 19:07:49
categories: Ruby Development
featured: false
tags: ruby
subclass: 'post tag-fiction'
categories: 'casper'
navigation: True
---

In ruby you can define methods at runtime, to do that, you can use for example the `define_method`  to create at runtime new instance methods.

<!--more-->

## Why would that be useful?

A classic use is to avoid code repetition. Imagine you have a class that is a basic mixin to help you to make external http requests.

{% highlight ruby %}
module Request
  def get(url, payload: {}, headers: {})
    headers = define_headers(headers)
    execute(:get, url, payload, headers)
  end

  def post(url, payload: {}, headers: {})
    headers = define_headers(headers)
    execute(:post, url, payload, headers)
  end

  def put(url, payload: {}, headers: {})
    headers = define_headers(headers)
    execute(:put, url, payload, headers)
  end

  def delete(url, payload: {}, headers: {})
    headers = define_headers(headers)
    execute(:delete, url, payload, headers)
  end

  def define_headers(headers)
    default_headers = {
      'Authorization' => "Bearer token",
      'Accept' => 'application/json'
    }
    default_headers.merge(headers)
  end

  private

  def execute(method, url, payload, headers)
    RestClient::Request.execute(method: method, url: url, payload: payload, headers: headers)
  end
end
{% endhighlight %}

Can you see how much code are we repeating just because of one single parameter?

One way to avoid that is using `define_method` to create the methods at runtime. Take a look at the following code:

{% highlight ruby %}
module Request
  [:get, :post, :put, :delete].each do |verb|
    define_method verb do |url, payload: {}, headers: {}|
      headers = define_headers(headers)
      execute(verb, url, payload, headers)
    end
  end

  def define_headers(headers)
    default_headers = {
      'Authorization' => "Bearer token",
      'Accept' => 'application/json'
    }
    default_headers.merge(headers)
  end

  private

  def execute(method, url, payload, headers)
    RestClient::Request.execute(method: method, url: url, payload: payload, headers: headers)
  end
end
{% endhighlight %}


As you can see, we just have to pass the method name as a parameter, and then, in the block definition you pass what will be your method's parameters.
