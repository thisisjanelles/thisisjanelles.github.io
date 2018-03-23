---
layout: post
title: Using Azure Functions to Call Azure Custom Vision Service (Part 2 - 3)
tags: [technical]
---

Last Monday, my team at work had a little free time and decided to hold an internal 3-day mini-hackathon to learn more about some Azure services that we were unfamiliar with. For this hackathon, our goal was basically to build a Progressive Web App that would use the local camera to capture an image and analyze it with a trained [Azure Custom Vision](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/) model that will recognize whether the image is a car or not.

This series of posts will go over the parts that I focused on during our mini-hackathon: training a model using the Azure Custom Vision Service and creating an [Azure Function](https://azure.microsoft.com/en-us/services/functions/) that connects to the Custom Vision Service to analyze an image.

I will also be including an extra feature we added at the end where the Azure Function we created will now also push records containing the image and analysis result into [CosmosDB](https://azure.microsoft.com/en-us/services/cosmos-db/), however, this will be less detailed since I didn't work on it directly and only added the code after my teammate completed it.

## Using Azure Functions to call Azure Custom Vision Service