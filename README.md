# Huginn for Newsrooms

[Huginn](https://github.com/cantino/huginn/) is a system for building agents that can automate many common web tasks. Think of it as an open source, more flexible version of IFTTT or a successor to Yahoo Pipes. It can let users quickly create simple bots or schedule and repeat routine tasks.

At The New York Times we've been [using it](https://source.opennews.org/en-US/articles/open-source-bot-factory/) for a few years to automate a variety of routine tasks. A few tasks it could be used for: monitor a webpage and alert you when it changes; alert a Slack channel when the lead story on the homepage is changed; email a reporter when their story is published by the CMS; regularly scrape a database and save it's results; and more.

* [How to Install](#how-to-install)
* [Creating an account](#creating-an-account)
* [Import a scenario](#list-of-scenarios)
* [Configure any credentials necessary](#user-credentials)
* [Go back to each agent and edit as necessary](#modify-agents-after-importing)
* [Picking CSS selectors from page](#more-on-css-selectors)

### Creating an Account

There's no central Huginn website, each installation is maintained separately. If someone has already set-up Huginn for you, use the invitation code and URL they've provided you.

If you have just want to try it out, there is a demo public install maintained by @albertsun at https://mighty-retreat-48764.herokuapp.com which you can register for with the invitation code `try-huginn`.

If you are slightly comfortable with the command line, have a Heroku account, you can run your own [Huginn installation](#how-to-install).

### Basic Concepts

Huginn is built around the concept of "Agents". Each Agent does one discrete, very simple task. To accomplish more complicated things requires combining agents into workflows. For example, one agent to check a webpage on a schedule, and another agent to send an email with the results from the first agent.

When you first create your account, it will have no agents. To get started, the easiest way will be to import and modify a scenario. Scenarios are pre-arranged sets of agents already configured for a larger task. Select "Scenarios" from the top menu and then "Import Scenario".

### List of Scenarios

You can find Scenarios provided by other Huginn users, or we're currently providing several useful scenarios as gists. To use one of these, grab the JSON from the Scenario Definition link and either modify and upload it, or paste the link to use it directly.

[Scenario Definitions](https://gist.github.com/albertsun/7e5cffc84a450c7d587f05f9f5b6938e)

These are currently configured with random existing websites to be replaced.

* Watch a website for changes, then send an email  
    `WebsiteAgent -> EmailAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/website-email-notifier.json)  
    In this case, one of Donald Trump's position pages

* Watch a website for changes in text, then send a slack notification  
    `WebsiteAgent -> SlackAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/website-slack-notifier.json)  
    In this case, one of Donald Trump's position pages

* Scrape a website and save it's text to [Stevedore](https://github.com/newsdev/stevedore)  
    `WebsiteAgent -> PostAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/website-to-stevedore.json)  

* Follow an RSS feed, and send the full text of its items to [Stevedore](https://github.com/newsdev/stevedore)  
    `RssAgent -> WebsiteAgent -> PostAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/rss-full-text-scrape-to-stevedore.json)  
    In this case, scraping all EEOC press releases

* Filter the NYTimes Timeswire API and email stories matching a regex  
    `WebsiteAgent -> TriggerAgent -> EmailAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/timeswire-story-filter-email.json)  
    In this case, filtering for any politics stories

* Alert a Slack channel when the lead story on the homepage is changed
    `WebsiteAgent -> SlackAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/hp-top-story-to-slack.json)  

* Get a morning email with new stories from a particular author
    `WebsiteAgent -> TriggerAgent -> EmailDigestAgent`  
    [Scenario Definition](https://raw.githubusercontent.com/albertsun/huginn-newsroom-scenarios/master/scenarios/author-filter-morning-email.json)  
    In this case, following http://www.theverge.com/tag/fitness and filtering for stories by Lauren Goode


### Importing Scenarios

If you're comfortable editing JSON directly you may find it easier to modify the scenario before uploading, otherwise you can modify the agents through the web UI after importing.

Here are the agents for the Website Email Notifier scenario.

```
{
  "type": "Agents::EmailAgent",
  "name": "Website Email Notifier",
  "disabled": false,
  "guid": "9c61eccfd02f22c23d8fe461bbc46805",
  "options": {
    "subject": "The Website Changed",
    "headline": "Your notification:",
    "expected_receive_period_in_days": "2"
  },
  "propagate_immediately": true
}
```

In the EmailAgent, the options to change would be `subject` for the subject line of the email and `headline`, for what headline to include in the body of the email.

```
{
  "type": "Agents::WebsiteAgent",
  "name": "Website Scraper",
  "disabled": false,
  "guid": "f820bf7d25c0fc527280a0c05d905b1f",
  "options": {
    "expected_update_period_in_days": "2",
    "url": "https://example.com/some-url",
    "type": "html",
    "mode": "on_change",
    "extract": {
      "text": {
        "css": "div#css-selector",
        "value": "normalize-space(.)"
      }
    }
  },
  "schedule": "every_30m",
  "keep_events_for": 0,
  "propagate_immediately": false
}
```

Then in the WebsiteAgent, you would set the `url` option to the page you want to scrape, and then the `extract` hash with what elements on the page should be selected. Use a CSS selector in `css` and an XPath expression in `value` to select the value from the selected elements. If you just want full text, leave the value as `normalize-space(.)`. Set `schedule` to specify how frequently the website should be checked. Valid options are [here](https://github.com/cantino/huginn/wiki/Creating-a-new-agent#scheduling).

### User Credentials

In the options for some agents, you'll see a tag like this `{% credential slack_webhook %}`. In the [Liquid templating language](https://github.com/cantino/huginn/wiki/Formatting-Events-using-Liquid) that Huginn uses that means to use a user credential. It's a more secure way to configure agents with secret variables like passwords and authentication tokens.

To set up credentials for agents you've created or imported, click "Credentials" in the top menu, and then "New Credential".

If an agent has the `{% credential slack_webhook %}` credential for example you'll want to create a webhook integration in Slack, copy the webhook URL from there and create a credential like so.

```
Credential name:    slack_webhook
Credential value:   https://hooks.slack.com/services/TKTK...YOURVALUE...TKTK
```

### Modify Agents after Importing

After importing the agents, you can continue modifying them and adding new agents as you see fit. Go through and replace whatever URLs and options are needed to fit the site you want to scrape or create more complicated configurations.

The [Huginn wiki](https://github.com/cantino/huginn/wiki/) has more examples of agent configurations Can make more extensive configuration changes following the documentation here and a guide to the [Liquid event templating language](https://github.com/cantino/huginn/wiki/Formatting-Events-using-Liquid) that allows variables to be inserted.

For example, with an Email Agent it will by default send emails to whatever email address you registered with. If you want to send email to a different address, go to edit the agent and add a `recipients` option with the email address (or multiple addresses as an array) which should receive the email.

Every agent will have a section of documentation on create or edit page explaining the options it uses.

### More on CSS Selectors

One of the most common and trickiest tasks for using Huginn is finding how to select the proper parts of pages for the WebsiteAgent. This requires some comfort and understanding of the DOM tree to be able to pick out the right nodes.

There's more documentation in the Huginn [WebsiteAgent](https://github.com/cantino/huginn/blob/master/app/models/agents/website_agent.rb#L38) page. One recommended way to find the right selectors is the [SelectorGadget](http://selectorgadget.com/) bookmarklet. Or using Firebug or your browser's built in developer tools.


## How to Install

If you don't already have an instance of Huginn you can use, the easiest way to deploy is with the one-click [Heroku installer](https://github.com/cantino/huginn/#heroku).

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/cantino/huginn)

* Click to deploy the app
* Run the command line `bin/setup_heroku` to configure an admin user, domain
* Either use the generated invite code or set a new one from
* Set up a sendgrid add-on to be able to send email

[Full Huginn Heroku documentation](https://github.com/cantino/huginn/blob/master/doc/heroku/install.md)

The main Huginn documentation is written for develoeprs and there are many other [installation options](https://github.com/cantino/huginn/#getting-started).
