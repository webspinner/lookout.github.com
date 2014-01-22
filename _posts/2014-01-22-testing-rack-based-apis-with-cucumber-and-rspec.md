---
layout: post
title: "Testing Rack-based APIs with Cucumber and RSpec"
tags:
- sinatra
- ruby
- rack
- testing
- gems
- cucumber
- rspec
---

Yesterday, I mentioned that we'd recently released
[`lookout-rack-test`](https://github.com/lookout/lookout-rack-test).  Today I'm
going to talk about how to use it for testing Rack-based JSON APIs. One of the
things we like to use this gem for is writing Cucumber scenarios as our **API
specification**, making sure that everybody agrees on what an API looks like
before we start implementing.


# Specifying API behavior with Cucumber

Before using Lookout::Rack::Test::Cucumber, you need to tell it where to find
your app:
```ruby
  Lookout::Rack::Test.app = TestApp
  require 'lookout/rack/test/cucumber'
```

This sets us up to be able to hit your routes, because we've included its routes
in our `World` object.

The key to the step definitions in Lookout::Rack::Test::Cucumber is the requests
- you can GET/PUT/POST/DELETE a route, of course:
```ruby
  Scenario: I should be able to GET a route, and check the response status
    When I GET "/"
    Then the response should be 200
```

 but we've also defined steps that let you do a request with a JSON body or a
table of parameters:
```ruby
  Scenario: POST with JSON body
    When I POST to "/json" with the JSON:
    """
      {
        "some_key" : "some_value",
        "another_key" : "another_value"
      }
    Then the response should be 201

  Scenario: POST with params
    When I POST to "/" with:
      | Name       | Value      |
      | this_param | this_value |
      | that_param | that_value |
    Then the response should be 201
```

Since we're talking about testing APIs, it's likely the response body will be
JSON, so we have a step for that as well:

```ruby
  Scenario: Response body containing JSON
    When I GET "/json"
    Then the response should be 200
    And I should receive the JSON:
    """
      { "key" : 1 }
    """
```

And for redirects:
```ruby
  Scenario: Redirect response
    When I GET "/redirect"
    Then I should be redirected to "/"
```

Finally, and perhaps most importantly, we have a module method `template_vars`
that points to the instance variable of the same name.  We use `template_vars`
to fill [Liquid templates](http://liquidmarkup.org/) in params, URIs, and JSON.
As such, we can do things like have a method that factories up some data, and
refer to that data in our tests.  In the following test, the step `I have
a user` might factory up a user and put its id and state values into
`template_vars['user_id']` and `template_vars['user_state']`, respectively:
```ruby
  Scenario: Liquid templates
    Given I have a user
    When I GET "/users/{% raw %}{{user_id}}{% endraw %}"
    The response should be 200
    And I should receive the JSON:
    """
      {
        "id" : {% raw %}{{user_id}}{% endraw %},
        "state" : "{% raw %}{{user_state}}{% endraw %}"
      }
    """
```

You can also combine this with capture groups in your step definitions.  For
instance, suppose that we want our `/users/:user_id` route not to return users
with the `archived` state.
```ruby
  When /^I have a user with the (.*) state$/ do |state|
    user = FactoryGirl.create :user, :state => state
    template_vars['user_id'] = user.id
    template_vars['user_state'] = user.state
  end
```
would allow multiple tests relating to the user's state:
```ruby
  Scenario: New user
    Given I have a user with the new state:
    When I GET "/users/{% raw %}{{user_id}}{% endraw %}
    Then the response should be 200
    And I should receive the JSON:
    """
      {
        "id" : {% raw %}{{user_id}}{% endraw %},
        "state" : "{% raw %}{{user_state}}{% endraw %}"
      }
    """

  Scenario: Archived
    Given I have a user with the archived state:
    When I GET "/users/{% raw %}{{user_id}}{% endraw %}
    Then the response should be 404
```

# Deeper API testing with RSpec

Like with Cucumber, the first thing you want to do is set up the application and
models:

```ruby
require 'lookout/rack/test'
require 'lookout/rack/test/rspec'
include Lookout::Rack::Test::RSpec
setup_models(TestModels)
setup_routes(TestApp)
```

`setup_models` assumes that the `Models` object passed to it has a `.setup` and
`.unsetup` method, which will be run around your test examples marked `{% raw %}:speed => :slow{% endraw %}`:

```ruby
  describe 'TestModels`, :speed => :slow do
    context 'A model with an id' do
      subject { described_class.new }

      it { should respond_to :id }
    end
  end
```

These examples will have factories loaded for them, and will setup and teardown
the models for each example.


`setup_routes` simply requires that it be passed a class that defines a Rack
application, something it can call `.new` on.  Having done that, you can use the
usual `Rack::Test::Methods` in tests of `:type => :route`:
```ruby
  describe '/api/public/v1/my_route', :type => :route do
    describe 'GET'
      subject(:response) { get "/api/public/v1/my_route" }

      its(:status) { should be 200 }
      its(:body) { should_not be_empty }
      it 'should do something else' do
        response
        expect(Model.count).not_to be 0
      end
    end
  end
```

Ensuring that different sets of examples are tagged with `:slow` or `:route`
makes it *much* easier to have some RSpec examples which run **very** fast, and
some RSpec examples which perform slower, more integration-level testing.


---


Hopefully you find `lookout-rack-test useful in testing and documenting your own Rack-based JSON APIs!

*- [Ian Smith](https://github.com/ismith)*
