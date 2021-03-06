---
title: Implement global message handlers | Microsoft Docs
description:  Learn how to enable your bot to listen for and handle user input containing certain keywords using the Bot Builder SDK for .NET.
author: kbrandl
ms.author: kibrandl
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 08/04/2017
ms.reviewer:
---

# Implement global message handlers

[!include[Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## Listen for keywords in user input

The following walk through shows how to implement global message handlers by using the Bot Builder SDK for .NET.

First, `Global.asax.cs` registers `GlobalMessageHandlersBotModule`, which is implemented as shown here. 
In this example, the module registers two scorables: one for managing a request to change settings (`SettingsScorable`) 
and another for managing a request to cancel (`CancelScoreable`).

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

The `CancelScorable` contains a `PrepareAsync` method that defines the trigger: 
if the message text is "cancel", this scorable will be triggered.

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

When a "cancel" request is received, the `PostAsync` method within `CancelScoreable` 
resets the dialog stack. 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

When a "change settings" request is received, the `PostAsync` method within `SettingsScorable` 
invokes the `SettingsDialog` (passing the request to that dialog), thereby adding the `SettingsDialog` 
to the top of the dialog stack and putting it in control of the conversation.

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## Sample code

For a complete sample that shows how to implement global message handlers using the Bot Builder SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Global Message Handlers sample</a> in GitHub.

## Additional resources

- [Design and control conversation flow](../bot-design-conversation-flow.md)
- <a href="https://docs.microsoft.com/en-us/dotnet/api/?view=botbuilder-3.8" target="_blank">Bot Builder SDK for .NET Reference</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Global Message Handlers sample (GitHub)</a>
