---
layout: post
title: I18n testing with Cucumber
tags:
- cucumber
- i18n
- localization
- atdd
---

Earlier this year we announced [support for various
languages](http://blog.mylookout.com/blog/2012/05/15/lookout-russian-korean-traditional-chinese/)
in our products, but we haven't discussed any of the engineering work
that went into the effort.

In order to localize our main Rails application powering
[mylookout.com](https://www.mylookout.com) we wanted to make sure that we were
correctly localizing many of the user flows that we already test with [Cucumber
and
Capybara](https://github.com/saucelabs/sauce_ruby/wiki/Cucumber-and-Capybara).

In this post I'll detail some of the challenges we faced in our localization
testing and the solutions we implemented.

### 1. Stringy Cucumber

It is not entirely uncommon to see Cucumber scenarios which reference very
specific strings in the UI, e.g.:

    Scenario: Log in to the site with valid credentials
      Given I am a registered user
      And my name is "Jonas"
      When I log in
      Then I should be greeted with "Welcome Jonas!"

The last step would be implemented with the following step definition:

    Then /^I should be greeted with "([^"]*)"$/ do |greeting|
      page.should have_content(greeting)
    end

This means that this scenario is going to fail 100% of the time if the user's
configured locale is German. Instead of seeing "Welcome Jonas!" the page would
render "Wilkommen Jonas!" and the scenario will fail.

There are really two issues at hand here to be addressed:

1. **Hard-coding UI text into Cucumber scenarios**

    When text is hard-coded into a scenario in this fashion, it makes the test
    more brittle not just to localization changes, but also marketing or product
    teams updating copy in the web application. If anybody updates a strings
    file and forgets about the Cucumber scenario, tests will all of a sudden start
    failing.


1. **Asserting page content based on hard-coded text**

    It's generally a good practice to check for specific CSS or XPath selectors
    instead of text, since the asserting page content based on the text can be
    slower and more brittle. If it can't be avoided, let's say if you're
    checking for a specific error message, then there are ways to make the check
    localizable which is covered below.

### 2. Localized assertions

There are valid cases where you cannot avoid asserting that a specific
message is displayed to the user. Since your Cucumber scenarios are running in
the same general environment that your Rails application, you can access the
same `I18n` methods that your controllers and views can access.

Let's take the scenario above, and update it a bit to make it easier to test
with localization:

    Scenario: Log in to the site with valid credentials
      Given I am a registered user
      And my name is "Jonas"
      When I log in
      Then I should be greeted

Then we'll change our step definition to check for a localized string:

    Then /^I should be greeted$/ do
      page.should have_content(I18n.t('dashboard.welcome', :name => first_name))
    end

Now we're validating that the page is using the right localized string key
"`dashboard.welcome`" and passing in our `first_name` variable. If this user
uses a German localization, then we'll be checking that they have the right
welcome message for their locale.


### 3. Testing around the world

By far the hardest challenge faced was testing the various languages. The first
step for us was running all of the scenarios using the different locales as the
default.

Notice in the scenario used as an example above, we never explicitly stated the
locale that the user would have, we naturally assume "en" is going to be Jonas'
locale, but it's not specified.

Whenever we create a user, we use the "default locale" of "en" with the ability
to override the locale with an environment variable, e.g.

    Given /^I am a registered user$/ do
      # Call a remote API to create a randomly generated user
      user = create_user(:type => :free,
                         :locale => ENV['CUCUMBER_LOCALE'] || 'en')

      # Hold onto this user object for future steps
      current_user = user
    end

This allows us to run the entire test suite with German users by simply
invoking Cucumber with:

    % CUCUMBER_LOCALE=de cucumber

---

Now we're running scenarios with German localizations, or so we hope, but how
can we check to make sure that the page has the correct translations? In some
cases we can check specific strings as mentioned above, but it is impractical
to do that for *every* string we render.

Instead, we want to make sure we're just not missing translations, which
required a custom [Capybara](https://github.com/jnicklas/capybara) driver to
check the page after actions.

Since some content might be rendered after an onclick action or other on-page
event, we've hooked the Capybara Selenium driver to run a few checks after
certain actions. The hooked Selenium driver itself can [be found in this
gist](https://gist.github.com/3764502), and then configure it properly with:

    Capybara.register_driver :selenium do |app|
        # Using a custom http client for performance reasons
        http_client = Selenium::WebDriver::Remote::Http::Default.new
        http_client.timeout = 120

        # Create a new driver object
        HookedSelenium::Driver.new(app, :http_client => http_client)
    end

To trigger the driver's assertions, we use the `CHECK_I18N` environment
variable, making our cucumber invocation look like:

    % CUCUMBER_LOCALE=de CHECK_I18N=1 cucumber

During this run of Cucumber, all users will be created with the "de" locale by
default, and the tests will raise an exception if we find anything that looks
like a missing translation (see [line 55](https://gist.github.com/3764502#L55)
of the gist above).


---

A combination of the approaches detailed above have allowed us to continue to
ensure the scenarios we write in Cucumber are portable across locales, as well
as making sure we continue to properly support a plethora of languages in our
Rails applications.


\- [R. Tyler Croy](https://github.com/rtyler/)
