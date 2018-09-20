---
layout: post
title: Creating a Basic Discord Bot Using Azure Bot Service (C#)
tags: [technical]
---

A few months back, as I was getting more deeply involved in the community of an FPS game I was playing, I was introduced to an app called [Discord](https://discordapp.com/). I thought Discord would just be another typical messaging app but it actually turned out to be so much more. I won't go into the details of why I think Discord is so awesome but, since then, Discord has been my most regularly used app and the main method that I use to talk to friends as well as engage in the communities I'm a part of.

Coincidentally, I had recently started to play around with [Azure Bot Service](https://azure.microsoft.com/en-ca/services/bot-service/) and had some experience through workshops/small hackathons that involved working with bots. However, all of the bot workshops I had been to only covered adding the bot to featured channels such as Skype.

So, naturally, I wanted to try and use Azure Bot Service to create a Discord bot. Spoiler: it wasn't as simple as I thought. I ran into several challenges, mostly with the fact that Discord was not a featured channel in Azure Bot Service so it wasn't as easy as just clicking a few buttons to add the bot. There were also some additional steps I had to figure out on Discord's end as well so this mini-project took longer than expected.

In this post, I'll be going through how I eventually got my very basic Discord bot to work and hopefully, this will be helpful to anyone wanting to use Azure Bot Service for their Discord bot!

## Create a Discord Application

1. Go to this [link](https://discordapp.com/developers/applications/me) and log in to your Discord account. Then click on **New App**.

Fill in the fields - only the app name is required - then click **Create App**.

Take note of the **Client ID** and **Client Secret** for later!

client id: 451424901153423370
client secret: u-OA1Lb83tIhw77MPWy1b_Vqtc8CRvCR

Scroll down and click **Create a Bot User**
Take not of the **Token** - NDUxNDI0OTAxMTUzNDIzMzcw.DfbsaA.DOx4laejeWCeOijJUmjbWuip7ME


## Visual Studio 2017 Setup

1. Install the Bot Builder SDK for .NET. 
*In Solution Explorer, right-click the project name and select Manage NuGet Packages....
*On the Browse tab, type "Microsoft.Bot.Builder" into the search box.
*Select Microsoft.Bot.Builder in the list of results, click Install, and accept the changes.

2. Create a new Bot Application Project
Click File
New Project
Search for and select **Bot Application**
Configure the fields as needed and click **OK**

## Publish

1. Create a Web App Bot resource in the Azure Portal
Go to the [Azure Portal](https://ms.portal.azure.com).
Click **Create a resource** and search for "bot".
Select **Web App Bot** and click **Create**.
Enter in a name for your bot.
Select the subscription you want to use.
Choose to use an existing or create a new resource group.
Choose a location close to you and a pricing tier (I just use the free one, **F0**, here!).
Give the app a name.
Set the bot template to **Basic C#**, configure a new App service plan, and choose to use an existing or create a new storage account.
Click **Create**

Go to the Web App Bot resource you just created.
Under **App Service Settings** in the left menu, click on **Application Settings** then scroll down to **App Settings**.
Take note of the **BotId**, **MicrosoftAppId**, and **MicrosoftAppPassword** (we'll need it later!)

2. Publish from Visual Studio
Go back to Visual Studio and, in Solution Explorer, right click your project then click **Publish**
In the Publish Target dialog that pops up after, make sure **App Service** is selected among the left menu items.
Select **Select Existing** under Azure App Service then click **Create Profile**.

If you're not logged in to your Microsoft account yet then log in or add an account from the dropdown in the top right corner of the dialog.
Choose the subscription the Web App Bot you created is in.
Navigate through the Resource Groups displayed until you reach your bot app.
Click **OK** (this may take a minute!)

On the following screen, click on **Settings**.
Click **Settings** again in the left menu and check **Remove additional files at destination**.
Click **Save** then, when the dialog closes, click **Publish**.

Once it's finished, click **Publish**
Your browser should open with the image below - success!

Navigate to the **Web.config** file in the project folder and under **<configuration>** and **<appSettings>**, add in the **BotId**, **MicrosoftAppId**, and **MicrosoftAppPassword** from earlier.

# Add Channel

Right click solution
**Manage NuGet Packages for Solution...**
Search Discord.Net
Tick checkbox of your project on the right
Click **Install**

Add bot to Discord server channel - https://discordapp.com/oauth2/authorize?client_id=451424901153423370&scope=bot&permissions=1

Go back to the Azure Portal and select your Web App bot.
Under **Bot Management** in the left menu, click **Channels**.

~~~
public class NoSqlRecord
    {
        public string id { get; set; }
        public string imageFileURL { get; set; }
        public string isCar { get; set; }
    }
~~~

imageFileURL with a test value.