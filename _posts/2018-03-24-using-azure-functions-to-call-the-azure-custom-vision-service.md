---
layout: post
title: Using Azure Functions to Call Azure Custom Vision Service (Part 2 - 3)
tags: [technical]
---

Following my last [post](https://thisisjanelles.github.io/2018-03-22-training-an-azure-custom-vision-service-model-to-recognize-a-car-image-part-1-3/) on training an [Azure Custom Vision Service](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/) model to recognize a car image, this second post in this series will focus on the next step of our mini-hackathon: creating a function using [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) that would call the Custom Vision Service to analyze an image and return to us whether the image is of a car or not.

## Using Azure Functions to call Azure Custom Vision Service