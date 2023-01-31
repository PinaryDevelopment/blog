---
layout: post
permalink: /architecture/the-power-of-micro-projects/micro-projects/1/data-model-exploration-and-blob-storage
title: "The Story Behind the Application - Micro-project 1"
excerpt: 'Micro-project #1 in the series explores iterating on the utilization of Azure storage for the audio files and exploring some of the data modeling needed for the project.'
categories: ['C#', 'micro-projects', 'architecture', 'patterns']
---

<aside>Starting any software development task has the potential to be daunting. There are a million details to be worked out, from business logic to technical details, implementation details to validation that all of the above works in tandem as expected. The process of going from ideation to delivery can be daunting. I'm taking a number of posts to walk through the development process I used to create a product from scratch. Attempting to discuss each stage of the process and document the approach I took to iteration and delivery.</aside>

*In the previous post, I started walking through the impetus behind the application and the progression of the first phase of the project. To recap, the goal of the application was to provide a site for a non-profit organization that delivers a lot of classes to the public. Due to the nature of the classes and the fact that there was a large long-running series that was starting soon, there was a desire to have a web app to go along with the series to enable attendees to catch up on classes that they missed or get an introduction to the material outside of the scheduled in-person lecture.*

> This series of posts is intended to walk through the development lifecycle I took for this application as well as to document different approaches to software design and development that I've utilized along the way. One piece that I feel has been very important to my approach to software development, both professionally and personally is the power of what I like to refer to as the 'micro-project'. In the introduction, I mentioned three micro-projects I utilized for the initial delivery of this application. I would like to focus on each one a bit and discuss the explorations/benefits of each.

Micro-projects have been a very beneficial approach for me for a number of reasons. One is that it helps me break down problem spaces into small bite-sized chunks. While this application wasn't meant to be a huge project, even the initial deliverable had a number of things that were unknown to me when I started it. One problem I knew I would need to solve was a way/means to store all of the audio files.

# Micro-project #1
As mentioned in the previous post, I'm familiar with Azure as a cloud provider and had settled on Azure blob storage as the means of storing all of the audio files. I knew that the initial iterations on the app were going to be rough and raw. So I wanted to test out the ability to connect to Azure storage and upload a file to it. I also wanted to organize the files in storage in a way that I would easily be able to find them if/when something went wrong.

> As this project is now three years old, the packages referenced and related code aren't current. They are using previous(now deprecated) sdks/apis. I believe that they are fine to share still though as this is about the process of development and not showing the current way of making this specific functionality work. The code has been updated to the latest SDKs in the current codebase.

I decided to use WinForms to create a small application that I would use initially for uploading the new files to blob storage. While setting up the UI and the event handling of a WinForms app can mostly be done via the UI provided in Visual Studio, I'm not familiar with how asynchronicity works in WinForms events. As this fact isn't critical to the functioning of the application and I didn't want to deep dive or get sidetracked by trying to learn that and the fact that the SDK's all provide asynchronous methods, I call each method in a synchronous fashion by appending `.ConfigureAwait(false).GetAwaiter().GetResult()` to the end of each invocation that returns a `Task`.

I could've put all of the asynchronous code in one method and would've only had to have done the above once, but I just wanted it working, so I didn't do any method extraction, but used one long method instead. The relevant code looked as follows:

{% highlight csharp linenos=table %}
var cloudBlobClient = CloudStorageAccount.Parse(ConfigurationManager.ConnectionStrings["AzureStorage"].ConnectionString).CreateCloudBlobClient();
var containerReference = cloudBlobClient.GetContainerReference("lectures");
containerReference.CreateIfNotExistsAsync().ConfigureAwait(false).GetAwaiter().GetResult();

static string createPathPart(string str) => str.ToLowerInvariant().Replace(" ", string.Empty);
var cloudFilePath = Path.Combine("src", createPathPart(author.Fullname));
foreach (var grouping in lecture.Groups)
{
    cloudFilePath = Path.Combine(cloudFilePath, createPathPart(grouping));
}
cloudFilePath = Path.Combine(cloudFilePath, $"{lecture.Topic}-{lecture.Version}-{date.ToString("yyyy.MM.dd")}{Path.GetExtension(originalFileName).ToLowerInvariant()}");
var blockBlobReference = containerReference.GetBlockBlobReference(cloudFilePath);
blockBlobReference.UploadFromStreamAsync(fileDialog.OpenFile()).ConfigureAwait(false).GetAwaiter().GetResult();
{% endhighlight %}

A brief explanation of the preceding code is as follows:
1. Get the configuration necessary to create a connection to Azure storage.
2. Get a reference to the container that will contain all of the files for the organization. *Each storage account can have multiple containers. These are the partitions used within a storage account.* I chose to create a container named specifically for the organization, so that I could utilize the application to support additional organizations in the future and have the partitioning make sense.
3. Create the filename, including the virtual directory it is contained within. *Inside of the containers, there aren't really directories. Each blob is just a filename and its contents. If the filename includes a `/` then blob storage creates virtual directories to represent that segmentation even though in actuality that segmentation doesn't exist.* More information about naming restrictions can be found in [Microsoft's docs](https://learn.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata). As can be seen or assumed from the above though, I own all of the data at this point, nothing is user entered from a webpage, so I skipped any sort of name validation or enforcement and just utilized the hard-coded data I created. I was careful with the data that was created to not include any characters (like an `'`)that might be an issue with Url encoding or Azure's restrictions in order to move quickly on the project and avoid errors.
4. Get a reference to the actual blob(this is in the container, which in turn is in the storage account). A blob is the actual file contents as well as the metadata associated with said file.
5. Upload the file.

And that was it...

While small and simple, it was a very impactful first step.

### Gained focus
Through this initial micro-implementation, I was able to focus on one aspect of the broader problem to solve. In doing so I had to:
1. Work out how to connect to Azure blob storage.
2. Determine what the correct connection string format was.
3. Learn the different Azure blob storage concepts.
4. Understand the correct order in which to invoke all of the methods.
5. Figure out how to craft a valid filename
6. Successfully upload the file(use a stream or a bytearray?, etc).

I was also able to iterate quickly on the format of the filename and the virtual directories associated with each one to get a feel for which felt more intuitive for me to maintain long-term. Azure blob storage's UI has the ability to 'search' on a blob name, but the search is really a 'starts with' filter. This realization led me to iterate on the best virtual directory structure and filename for quickly locating files that seemed to have processed incorrectly. I was also able to manually download the file from storage post-upload to verify that my implementation didn't create and corruption in the file itself.

### Reduced distractions
In my experience, the ability to narrow down the problem space and focus on small parts of the larger problem/implementation at a time, really enables cleaner, clearer code. Creating a ton of abstraction from the get go or trying to anticipate every eventual possibility, often leads to overly complicated, confusing, hard to read/hard to debug code. In utilizing the approach above, even for this small problem, I was able to create a small application with a UI. The UI was created via drag and drop in Visual Studio. It won't win any design awards, but it was easy for me to launch everyday and easy for me to upload new lectures. This limited the potential for typos and allowed me to iterate quickly.

I was able to ignore a number of things that could've been distractions from this core problem, such as:
1. I was able to utilize `ConfigurationManager` to do so and didn't have to worry about how Azure functions deals with configurations/connection strings.
2. I ignored the retrieval of files.
3. I didn't need to concern myself with the user interface and the validation concerns that would be much more relevant if the data was coming from the user.

With proof of concept one out of the way, I had something of value, that took me one step closer to my initial launch goal and allowed me to move on to problem #2, namely utilizing an Azure function to perform file manipulation to reduce the sampling size and file size of each audio file.
