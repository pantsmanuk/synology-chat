# Synology Chat Notifications Channel for Laravel

[![License](https://img.shields.io/badge/License-BSD_3--Clause-brightgreen.svg?style=flat-square)](LICENSE)

This package makes it easy to send notifications using [Synology Chat](https://www.synology.com/en-global/dsm/feature/chat) with Laravel 11.x and above

```php
return SynologyChatMessage::create()
    ->to(config('services.synology_chat.sales_url'))
    ->type('success')
    ->title('Subscription Created')
    ->content('Yey, you got a **new subscription**. Maybe you want to contact him if he needs any support?')
    ->button('Check User', 'https://foo.bar/users/123');
```
## Contents

- [Synology Chat Notifications Channel for Laravel](#microsoft-teams-notifications-channel-for-laravel)
  - [Contents](#contents)
  - [Installation](#installation)
    - [Setting up the Connector](#setting-up-the-connector)
    - [Setting up the MicrosoftTeams service](#setting-up-the-microsoftteams-service)
  - [Usage](#usage)
    - [On-Demand Notification Usage](#on-demand-notification-usage)
    - [Available Message methods](#available-message-methods)
      - [Sections](#sections)
  - [Changelog](#changelog)
  - [Testing](#testing)
  - [Security](#security)
  - [Contributing](#contributing)
  - [Credits](#credits)
  - [License](#license)


## Installation

You can install the package via composer:

``` bash
composer require laravel-notification-channels/synology-chat
```

Next, if you're using Laravel _without_ auto-discovery, add the service provider to `config/app.php`:

```php
'providers' => [
    // ...
    NotificationChannels\SynologyChat\SynologyChatServiceProvider::class,
],
```

### Setting up the Connector

Please check out [this](https://docs.microsoft.com/en-gb/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook#add-an-incoming-webhook-to-a-teams-channel) for setting up and adding a webhook connector to your Team's channel. Basic Markdown is supported, please also check out the [message card reference article](https://docs.microsoft.com/en-us/outlook/actionable-messages/message-card-reference#httppost-action) which goes in more detail about the do's and don'ts.

### Setting up the MicrosoftTeams service

Then, configure your webhook url:

Add the following code to your `config/services.php`:

```php
// config/services.php
...
'synology_chat' => [
    'webhook_url' => env('CHAT_WEBHOOK_URL'),
],
...
```

You can also add multiple webhooks if you have multiple teams or channels, it's up to you.

```php
// config/services.php
...
'synology_chat' => [
    'sales_url' => env('CHAT_SALES_WEBHOOK_URL'),
    'dev_url' => env('CHAT_DEV_WEBHOOK_URL'),
],
...
```
## Usage

Now you can use the channel in your `via()` method inside the notification:

```php
use Illuminate\Notifications\Notification;
use NotificationChannels\MicrosoftTeams\SynologyChatChannel;
use NotificationChannels\MicrosoftTeams\SynologyChatMessage;

class SubscriptionCreated extends Notification
{
    public function via($notifiable)
    {
        return [SynologyChatChannel::class];
    }

    public function toSynologyChat($notifiable)
    {
        return SynologyChatMessage::create()
            ->to(config('services.synology_chat.sales_url'))
            ->type('success')
            ->title('Subscription Created')
            ->content('Yey, you got a **new subscription**. Maybe you want to contact him if he needs any support?')
            ->button('Check User', 'https://foo.bar/users/123');
    }
}
```

Instead of adding the `to($url)` method for the recipient you can also add the `routeNotificationForSynologyChat` method inside your Notifiable model. This method needs to return the webhook url.

```php
public function routeNotificationForSynologyChat(Notification $notification)
{
    return config('services.synology_chat.sales_url');
}
```

### On-Demand Notification Usage


To use on demand notifications you can use the `route` method on the Notification facade. 

```php
Notification::route(SynologyChatChannel::class,null)
    ->notify(new SubscriptionCreated());
```


### Available Message methods

- `to(string $webhookUrl)`: Recipient's webhook url.
- `title(string $title)`: Title of the message.
- `summary(string $summary)`: Summary of the message.
- `type(string $type)`: Type which is used as theme color (any valid hex code or one of: primary|secondary|accent|error|info|success|warning).
- `content(string $content)`: Content of the message (Markdown supported).
- `button(string $text, string $url = '', array $params = [])`: Text and url of a button. Wrapper for an potential action.
- `action(string $text, $type = 'OpenUri', array $params = [])`: Text and type for a potential action. Further params can be added depending on the action. For more infos about different types check out [this link](https://docs.microsoft.com/en-us/outlook/actionable-messages/message-card-reference#actions).
- `options(array $options, $sectionId = null)`: Add additional options to pass to the message payload object.

#### Sections
It is possible to define one or many sections inside a message card. The following methods can be used within a section
- `addStartGroupToSection($sectionId = 'standard_section')`: Add a startGroup property which marks the start of a logical group of information.
- `activity(string $activityImage = '', string $activityTitle = '', string $activitySubtitle = '', string $activityText = '', $sectionId = 'standard_section')`: Add an activity to a section.
- `fact(string $name, string $value, $sectionId = 'standard_section')`: Add a fact to a section (Supports Markdown).
- `image(string $imageUri, string $title = '', $sectionId = 'standard_section')`: Add an image to a section.
- `heroImage(string $imageUri, string $title = '', $sectionId = 'standard_section')`: Add a hero image to a section.

Additionally the title, content, button and action can be also added to a section through the optional `params` value:
- `title(string $title, array $params = ['section' => 'my-section'])`: Title of the message and add it to `my-section`.
- `content(string $content, array $params = ['section' => 'my-section'])`: Content of the message and add it to `my-section` (Markdown supported).
- `button(string $text, string $url = '', array $params = ['section' => 'my-section'])`: Text and url of a button and add it to `my-section`.
- `action(string $text, $type = 'OpenUri', array $params = ['section' => 'my-section'])`: Text and type of an potential action and add it to `my-section`.

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

``` bash
$ composer test
```

## Security

If you discover any security related issues, please email murray.crane@ggpsystems.co.uk instead of using the issue tracker.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits
This channel is based on the Microsoft Teams notification channel. 

- [Tobias Madner](https://github.com/Tob0t)
- [All Contributors](../../contributors)

## License

The BSD 3-clause licence. Please see [License File](LICENSE) for more information.
