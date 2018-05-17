---
layout: post
title: Using Azure Functions to Call Azure Custom Vision Service and Connect to CosmosDB (Part 2 - 2)
tags: [technical]
---

Following my last [post](https://thisisjanelles.github.io/2018-03-22-training-an-azure-custom-vision-service-model-to-recognize-a-car-image-part-1-2/) on training an [Azure Custom Vision Service](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/) model to recognize a car image, this second post in will focus on the next step of our mini-hackathon: creating a function using [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) that would call the Custom Vision Service to analyze an image and return to us whether the image is of a car or not.

## Using Azure Functions to call Azure Custom Vision Service

My partner and I had no previous experience using Azure Functions so we decided to each try a different way of setting up our first basic functions by following the [Azure Quickstart documentation](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function). From this, we could start to modify the code as needed.

We started with creating the classes we would need and, to simplify, we kept everything in one file just so we wouldn't have to figure out how having several class files would transfer to Azure Functions.

~~~
public class Global
    {
        public string result;
    }

    public class Prediction
    {
        public string TagId { get; set; }
        public string Tag { get; set; }
        public double Probability { get; set; }
    }

    public class PayloadPrediction
    {
        public PayloadPrediction(RootObject rootObject)
        {
            this.IsCar = (rootObject.Predictions[0].Probability > 0.6) ? true : false;
        }

        public bool IsCar { get; set; }
    }

    public class RootObject
    {
        public string Id { get; set; }
        public string Project { get; set; }
        public string Iteration { get; set; }
        public DateTime Created { get; set; }
        public List<Prediction> Predictions { get; set; }
    }
~~~

For the function itself, we luckily didn't have to start completely from scratch and referenced some useful documentation from Microsoft on [how to use the Custom Vision Service prediction endpoint](https://docs.microsoft.com/en-us/azure/cognitive-services/custom-vision-service/use-prediction-api) to test images programmatically with the classifier we trained earlier.

Our function code is shown below with the modifications we made to the documentation sample code.

~~~
public class AutoFunction
    {
        [FunctionName("AutoFunction")]
        public static IActionResult Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequest req, TraceWriter log) {

            // Passing in the image URL, in our case, it was a url to a file uploaded in Azure Blob Storage (done by different members of our team)
            string imageFileURL = req.Query["imageFileURL"]; ;
            var webClient = new WebClient();
            byte[] imageBytes = webClient.DownloadData(imageFileURL);

            var result = MakePredictionRequest(imageBytes).Result;

            return imageFileURL != null
                ? (ActionResult)new OkObjectResult(result)
                : new BadRequestObjectResult("Please pass a image file URL on the query string or in the request body");
        }

        private static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async Task<string> MakePredictionRequest(byte[] imageByteArray)
        {
            var client = new HttpClient();
            var jsonResponse = string.Empty;
            // Request headers - replace with your valid subscription key.
            client.DefaultRequestHeaders.Add("prediction-key", "<subscription-key>");

            // Prediction URL - replace with your valid prediction URL.
            string url = "<prediction-URL";

            HttpResponseMessage response;

            // Request body. Try this sample with a locally stored image.
            using (var content = new ByteArrayContent(imageByteArray))
            {
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                response = await client.PostAsync(url, content);
                jsonResponse = await response.Content.ReadAsStringAsync();
                RootObject deserializedRootOjbect = JsonConvert.DeserializeObject<RootObject>(jsonResponse);
                PayloadPrediction payloadPrediction = new PayloadPrediction(deserializedRootOjbect);
                Console.WriteLine("Is Car = " + payloadPrediction.IsCar.ToString());
                string result = payloadPrediction.IsCar.ToString();
                return result;
            }
        }
    }
~~~

## Using Azure Functions to connect to CosmosDB

I didn't personally work on the connection to CosmosDB but I will include the code my partner worked on that added the functionality to the Azure Function we just created.

We first needed an additonal class for a record in CosmosDB.

~~~
public class NoSqlRecord
    {
        public string id { get; set; }
        public string imageFileURL { get; set; }
        public string isCar { get; set; }
    }
~~~

Using that class, we then added more code to the Run method. Unfortunately, we were not able to complete passing an actual image URL into the record within our time constraints so the code below shows imageFileURL with a test value.

~~~
Task.Run(async () =>{
    // Do any async anything you need here without worry
    var client = new DocumentClient(new Uri("<cosmos-db-service-endpoint>"), "<authKey");
    
    NoSqlRecord noSqlRecord1 = new NoSqlRecord
    {
        // Create a new unique GUID for each new record
        id = Guid.NewGuid().ToString(),
        imageFileURL = "Test with ID=TestFunction2NotCar",
        isCar = result
    };
        
        var x = await client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri("<database-id>", "<collection-id>"), noSqlRecord1);
    }).GetAwaiter().GetResult();

~~~

And now we've connected our Custom Vision function to CosmosDB! There's still a lot of things we could add and fix but our goal was to learn and, in the short time we were working on this mini-hack, I definitely learned a lot about how to work with the Azure Custom Vision Service and Azure Functions.

{: .box-note}
**Part 1:** [Training an Azure Custom Vision Service Model to Recognize a Car Image](https://thisisjanelles.github.io/2018-03-22-training-an-azure-custom-vision-service-model-to-recognize-a-car-image-part-1-2/)