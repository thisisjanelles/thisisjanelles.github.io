---
layout: post
title: Training an Azure Custom Vision Service Model to Recognize a Car Image (Part 1 - 2)
tags: [technical]
---

Last Monday, my team at work had a little free time and decided to hold a 3-day mini-hackathon to learn more about some Azure services that we were unfamiliar with. For this hackathon, our goal was to build a Progressive Web App that would use the local camera to capture an image and analyze it with a trained [Azure Custom Vision](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/) model that will recognize whether the image is a car or not.

This series of posts will go over the parts that I focused on during our mini-hackathon: training a model using the Azure Custom Vision Service and creating an [Azure Function](https://azure.microsoft.com/en-us/services/functions/) that connects to the Custom Vision Service to analyze an image.

I will also be including an extra feature we added at the end where the Azure Function we created will now also push records containing the image and analysis result into [CosmosDB](https://azure.microsoft.com/en-us/services/cosmos-db/), however, this will be less detailed since I didn't work on it directly and only added the code after my teammate completed it.

## Training an Azure Custom Vision Service model to recognize a car image

The first task we had to do was find a way to use Azure Cognitive Services to perform the image analysis.

My first thought in this situation was to use Microsoft's [Computer Vision API](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/) to return information about the image. This would have probably been a valid solution, however, the API would return a lot of information that were unnecessary for our hack's purpose. The sample response from the API below (taken from Microsoft documentation [here](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/quickstarts/csharp)) shows that the API would give us whatever categories and tags it thought was in the image as well as various metadata.

~~~
{
   "categories": [
      {
         "name": "abstract_",
         "score": 0.00390625
      },
      {
         "name": "others_",
         "score": 0.0234375
      },
      {
         "name": "outdoor_",
         "score": 0.00390625
      }
   ],
   "description": {
      "tags": [
         "road",
         "building",
         "outdoor",
         "street",
         "night",
         "black",
         "city",
         "white",
         "light",
         "sitting",
         "riding",
         "man",
         "side",
         "empty",
         "rain",
         "corner",
         "traffic",
         "lit",
         "hydrant",
         "stop",
         "board",
         "parked",
         "bus",
         "tall"
      ],
      "captions": [
         {
            "text": "a close up of an empty city street at night",
            "confidence": 0.7965622853462756
         }
      ]
   },
   "requestId": "dddf1ac9-7e66-4c47-bdef-222f3fe5aa23",
   "metadata": {
      "width": 3733,
      "height": 1986,
      "format": "Jpeg"
   },
   "color": {
      "dominantColorForeground": "Black",
      "dominantColorBackground": "Black",
      "dominantColors": [
         "Black",
         "Grey"
      ],
      "accentColor": "666666",
      "isBWImg": true
   }
}
~~~

The other option was to try and use the Azure Custom Vision Service, which is currently still in preview. The Custom Vision Service would allow us to customize our own computer vision model that would only return the specific information we needed.

Since the Custom Vision Service would have more concise results, was something we had never used, and was something that would be useful to know about for a use case where someone wanted a custom model, we decided to use the Custom Vision Service instead.

Getting started with Custom Vision was quite simple. I was first directed to the [website](https://customvision.ai/) where I logged in with my Microsoft account credentials (the same one with my Azure subscription).

After signing in, there's a simple interface to create a new project. Clicking **New Project** gives a dialog where you can enter the information for your project.

![New Project](/img/MiniHack%20Photos/new-project-screen.png)

From there, it was a simple matter of uploading images to train the classifier (more documentation [here](https://docs.microsoft.com/en-us/azure/cognitive-services/custom-vision-service/getting-started-build-a-classifier)). Initially I started to create different tags that identified different vehicles such as *car*, *cars* (for images with multiple cars), *bus*, and *truck*. For the hack's purposes, I modified this later to only have two tags: **car** and **notCar**. The **car** tag would only contain images of single cars while the **notCar** tag would be for everything else.

![Image Training](/img/MiniHack%20Photos/image-training.PNG)

However, I realized that there was no way for the Custom Vision Service to categorize everything that was not a car into **notCar**. Essentially, if you hadn't trained the model to identify a boat as **notCar**, it wouldn't be able to infer from the fact that it didn't match **car** to conclude that it should be placed into **notCar**.

This was a small problem as the way we had planned to program our Azure Function was to compare the percentage score that would be returned and if **car** > **notCar**, then our Function would set the result of **isCar** as true. However, because of this, there were some cases where the score of **notCar** actually ended up greater than **car** even though they were both quite low because of the model not being trained and confident of what to tag the particular image.

I tried to get over this by first just trying to train the model with as much random images as I could that would be tagged as **notCar**. This soon proved tedious and we later solved that problem programmatically in our Function instead.

![Performance Test](/img/MiniHack%20Photos/performance-test.PNG)

I trained the model until it achieved above 60%, a threshold we agreed on earlier. As seen below, the JSON response from the Custom Vision Service is a lot more concise. We were able to specify exactly what classification we needed so we would only see whether it was a car or not and not have to filter through multiple other tags that were irrelevant to us.

~~~
"Predictions":[  
    {  
        "TagId":"c0bf9dcf-09ed-448f-b956-71c4fe2c6f5e",
        "Tag":"notCar",
        "Probability":1.8441413E-05
    },
    {  
        "TagId":"70666965-666f-4da0-b8e5-e0e1fd8856de",
        "Tag":"car",
        "Probability":1.105892E-06
    }
]
~~~

After this, I obtained the Prediction URL, key, and other information by clicking on the **Prediction URL** button at the top of the screen. We would need this for the next part of the hack - creating an Azure Function to call the Custom Vision Service.

{: .box-note}
**Part 2:** [Using Azure Functions to Call the Azure Custom Vision Service and Connect to CosmosDB](https://thisisjanelles.github.io/2018-03-24-using-azure-functions-to-call-the-azure-custom-vision-service-and-connect-to-cosmosdb-c-part-2-2/)