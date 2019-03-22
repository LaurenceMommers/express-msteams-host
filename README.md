# Express utility for Microsoft Teams Apps

[![npm version](https://badge.fury.io/js/express-msteams-host.svg)](https://badge.fury.io/js/express-msteams-host)

This utility for [Express](https://expressjs.com/) is targeted for [Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/) applications built generated by the [Microsoft Teams Yeoman generator](https://aka.ms/yoteams) hosted on Express.

 | @master | @preview |
 :--------:|:---------:
 [![Build Status](https://travis-ci.org/wictorwilen/express-msteams-host.svg?branch=master)](https://travis-ci.org/wictorwilen/express-msteams-host)|[![Build Status](https://travis-ci.org/wictorwilen/express-msteams-host.svg?branch=preview)](https://travis-ci.org/wictorwilen/express-msteams-host)

The Middleware serves two major purposes:

* Automatic routing of web services for Bots, Connectors and Outgoing Webhooks based on TypeScript decorators
* Automatic routing of static pages for Tabs and Connectors

The middleware is automatically added to projects generated by the Microsoft Teams Yeoman generator.

## Usage

For the automatic routing to work the following usage MUST be followed.

### Creating a bot

### Implementation and decorator usage for Bots

Bots MUST be implemented as a class using the `IBot` interface and decorated using the `BotDeclaration` decorator.

``` TypeScript
import { BotDeclaration, IBot } from 'express-msteams-host';
import { MemoryStorage, ConversationState, TurnContext } from 'botbuilder';

@BotDeclaration(
    '/api/messages',
    new MemoryStorage(),
    process.env.MICROSOFT_APP_ID,
    process.env.MICROSOFT_APP_PASSWORD)
export class myBot implements IBot {

    public constructor(conversationState: ConversationState) {
        ...
    }

    public async onTurn(context: TurnContext): Promise<any> {
        ...
    }

}
```

### Decorators for Message Extensions

Message Extensions MUST be a class implementing the `IMessageExtension` interface and when declared in the bot implementation decorated with the `MessageExtensionDeclarator` decorator.

``` TypeScript
import { IMessageExtension } from 'express-msteams-host';
import { TurnContext } from 'botbuilder';
import { MessagingExtensionQuery, InvokeResponseTyped, MessagingExtensionResponse } from 'botbuilder-teams';


export default class MyMessageExtension implements IMessageExtension {
    public async onQuery(context: TurnContext, query: MessagingExtensionQuery): Promise<InvokeResponseTyped<MessagingExtensionResponse>> {
       ...
    }

    // this is used when canUpdateConfiguration is set to true 
    public async onQuerySettingsUrl(context: TurnContext): Promise<InvokeResponseTyped<{ composeExtension: { type: string, suggestedActions: { actions: Array<{ type: string, title: string, value: string }> } } }>> {
        ...
    }

    public onSettingsUpdate(context: TurnContext): Promise<InvokeResponse> {
       ...
    }

}
```

In the implementation of the bot, define the message extensions as below. You are responsible for instantiating the object, you might want to add additional parameters or configuration.

``` TypeScript
import { BotDeclaration, IBot } from 'express-msteams-host';
import { MemoryStorage, ConversationState, TurnContext } from 'botbuilder';
import { MyMessageExtension } from './MyMessageExtension';

@BotDeclaration(
    '/api/messages',
    new MemoryStorage()
    process.env.MICROSOFT_APP_ID,
    process.env.MICROSOFT_APP_PASSWORD)
export class myBot implements IBot {

    @MessageExtensionDeclaration('myMessageExtension')
    private _myMessageExtension: MyMessageExtension;

   public constructor(conversationState: ConversationState) {

        this._myMessageExtension = new MyMessageExtension();

        ...
    }

}
```

### Decorator for Connectors

Connectors MUST be implemented as a class implementing the `IConnector` interface and decorated using the `ConnectorDeclaration` decorator.

``` TypeScript
import { ConnectorDeclaration, IConnector } from 'express-msteams-host';
import { Request } from "express";

@ConnectorDeclaration(
    '/api/connector/connect',
    '/api/connector/ping',
    'web/myConnectorConnect.ejs',
    '/myConnectorConnected.html'
)
export class myConnector implements IConnector {
    Connect(req: Request): void {
        ...
    }

    Ping(req: Request): Promise<void>[] {
        ...
    }
}
```

### Decorator for Outgoing Webhooks

Outgoing Webhooks MUST be implemented as a class implementing the `IOutdegoingWebhook` interface and decorated using the `OutgoingWebhookDeclaration` decorator.

``` TypeScript
import { OutgoingWebhookDeclaration, IOutgoingWebhook } from 'express-msteams-host';
import * as express from 'express';

@OutgoingWebhookDeclaration('/api/webhook')
export class myOutgoingWebhook implements IOutgoingWebhook {
    public requestHandler(req: Express.Request, res: Express.Response, next: Express.NextFunction): any {
        ...
    }
}
```

## Logging

To enable logging from this module you need to add `msteams` to the `DEBUG` environment variable. See the [debug package](https://www.npmjs.com/package/debug) for more information.

Example for Windows command line:

> SET DEBUG=msteams

## License

Copyright (c) Wictor Wilén. All rights reserved.

Licensed under the MIT license.