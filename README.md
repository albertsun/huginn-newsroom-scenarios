# Huginn for Newsrooms

## How to Install

If you don't already have an instance of Huginn you can use, the easiest way to deploy is with the one-click [Heroku installer](https://github.com/cantino/huginn/#heroku).

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/cantino/huginn)

* Click to deploy the app
* Run the command line `bin/setup_heroku` to configure an admin user, domain
* Either use the generated invite code or set a new one from
* Set up a sendgrid add-on to be able to send email

[Full Huginn Heroku documentation](https://github.com/cantino/huginn/blob/master/doc/heroku/install.md)

## How to Use

* Create an account
* Import a scenario
* Configure any credentials necessary
* Go back to each agent and edit as necessary
* Picking CSS selectors from page

### Creating an Account

Signing up for an account should be straightforward. Either use the invitation code provided by whoever installed the app, or which you set up in Heroku.

If you have neither, there is currently a public install maintained by @albertsun at https://mighty-retreat-48764.herokuapp.com which you can register for with the invitation code `try-huginn`.

### Basic Concepts

Huginn is built around the concept of "Agents". Each Agent does one discrete, very simple task. To accomplish more complicated things requires combining agents into workflows. For example, one agent to check a webpage on a schedule, and another agent to send an email with the results from the first agent.

When you first create your account, it will have no agents. To get started, the easiest way will be to import and modify a scenario. Scenarios are pre-arranged sets of agents already configured for a larger task. Select "Scenarios" from the top menu and then "Import Scenario".

### List of Scenarios

You can find Scenarios provided by other Huginn users, or we're currently providing several useful scenarios as gists.

[Scenario Definitions](https://gist.github.com/albertsun/7e5cffc84a450c7d587f05f9f5b6938e)

These are currently configured with random existing websites to be replaced.

* Watch a website for changes, then send an email  
    `WebsiteAgent -> EmailAgent`  
    [Scenario Definition](https://gist.github.com/albertsun/7e5cffc84a450c7d587f05f9f5b6938e#file-website-email-notifier-json)  

* Watch a website for changes, then send a slack notification  
    `WebsiteAgent -> SlackAgent`  
    [Scenario Definition](https://gist.github.com/albertsun/7e5cffc84a450c7d587f05f9f5b6938e#file-website-slack-notifier-json)  

* website scrape to stevedore POST  
    `WebsiteAgent -> PostAgent`  
    [Scenario Definition](https://gist.github.com/albertsun/7e5cffc84a450c7d587f05f9f5b6938e#file-website-to-stevedore-json)  

* RSS feed full page scrape to stevedore POST  
    `RssAgent -> WebsiteAgent -> PostAgent`  
    [Scenario Definition](https://gist.github.com/albertsun/7e5cffc84a450c7d587f05f9f5b6938e#file-rss-full-text-scrape-to-stevedore-json)  

* alert a Slack channel when the lead story on the homepage is changed
* email a reporter when their story is published by the CMS
* detect a spike in twitter mentions of a story URL and send a notification


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

### Configure/Add Credentials

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
