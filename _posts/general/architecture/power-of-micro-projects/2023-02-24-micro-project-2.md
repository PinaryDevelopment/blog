---
layout: post
permalink: /architecture/the-power-of-micro-projects/micro-projects/2/audio-file-size-and-executables-in-azure-functions
title: "The Story Behind the Application - Micro-project 2"
excerpt: 'Micro-project #2 in the series explores reducing the size of audio files and executing an executable within an Azure function via C#.'
categories: ['C#', 'micro-projects', 'architecture', 'patterns', 'Azure', 'Azure functions']
---

<aside>Starting any software development task has the potential to be daunting. There are a million details to be worked out, from business logic to technical details, implementation details to validation that all of the above works, in tandem, as expected. The process of going from ideation to delivery can be daunting. I'm taking a number of posts to walk through the development process I used to create a product from scratch. Attempting to discuss each stage of the process and document the approach I took to iteration and delivery.</aside>

*In the previous post, I discussed the first micro-project I utilized to get this application started. It discussed my exploration around Azure blob storage and some of the data models I utilized to get the application off of the ground.*

> This series of posts is intended to walk through the development lifecycle I took for this application as well as to document different approaches to software design and development that I've utilized along the way. One piece that I feel has been very important to my approach to software development, both professionally and personally is the power of what I like to refer to as the 'micro-project'. In the introduction, I mentioned three micro-projects I utilized for the initial delivery of this application. I would like to focus on each one a bit and discuss the explorations/benefits of each.

# Micro-project #2
Launching off of the previous micro-project, I now knew how I was going to store the audio files. I wanted to delve into the manipulation of those files. *To be clear, this step was entirely unnecessary. The application could've taken the files sent by the lecturers and provided them directly to the attendees via a web interface. I took this detour for a couple of reasons.*

1. I believe it is worthwhile to be a responsible steward of the World Wide Web. We are all in this together and the more irresponsible we are with the applications we develop, the more everyone is effected.
2. As these lectures are publicly available and free of charge, there is a wide variety of people that participate. Some are in lower income brackets. I didn't want to assume that everyone utilizing the application had high-speed internet or unlimited data plans on their cellphones.
3. Since I was doing this on a volunteer basis, I wanted to utilize the opportunity to explore new vistas with the project and push myself technically.
4. Seeing that this step was completely unnecessary to achieve an entirely functional application. I knew that I could time-box this effort and if it wasn't working within x amount of time(in this case a week), I could set aside the work for a later date. As this approach would effect the pathing of files and have a few other downstream effects, it made sense though to try to have this functionality from day 1.

### Research problem space
The first step of this process was research. As I'd never worked with audio files before, I started searching around for how to make audio files smaller. There were some results around the compression algorithm used by the server to return the http response. While this would positively impact every request responded to by the server, not just the audio file responses, for the audio files specifically there was a minimal amount of savings.

Looking at the C# landscape, there was one library that seemed to be the main library for audio manipulation, [NAudio](https://github.com/naudio/NAudio). Looking at the docs and reading many articles by [the author](https://markheath.net/) of the library, this seemed very promising. The library is well documented and has many examples. There were also a few PluralSight courses on the library by the author and there just so happened to be a 'free weekend' on the horizon. After quite a bit of reading and watching, I'd learned quite a bit about how audio files are created and what some of the differences were between some of the main audio encodings. This area was fun to explore and helped me to understand why the mp3 files that I received were so much smaller than wav files of similar playback length(40MB vs 1GB).

Through this exploration as well, I understood that what would have a minimal impact on the audio quality of the files I was working with, but would effect a meaningful reduction of the end file size was the bitrate. In short, the higher the bitrate, the larger the file size and the higher the audio fidelity. BUT, the fidelity matters most when there is a lot of subtlety to the audio, i.e. music. When the audio is someone speaking, where there is really just one sound of interest and not a whole lot of variety in that sound, then the bitrate becomes less important. In fact, utilizing a bitrate of 32kbps effects an imperceptible change in the quality of the audio for lectures, whereas 320kbps would be a bitrate recommended more for music files.

### Attempt #1
I set out trying to learn how to reduce the sample rate with NAudio and...well, I couldn't get it to work. As I stated, there are some really good docs and walk-throughs out there on the library itself, but no matter what I tried, I couldn't get it to work. It seemed as though there were aspects of the library that relied upon certain executables or libraries that were Windows OS specific. So after a day of exploration on that front, I put the work with NAudio aside and decided to do a bit more research.

### Back to the drawing board
Knowing now the specific aspect of the data manipulation I wanted to focus on, I changed the focus of my search. I also tried to look into [Audacity](https://www.audacityteam.org/) a bit to see what they used under the hood. And that is when I stumbled upon [ffmpeg](https://ffmpeg.org/). It is cross-platform, is invocable via the cli and seemed to not only have what I was looking for currently, but so much more that could be utilized for additional functionality later on.

### Attempt #2
Seeing that this was an executable and not a library to be referenced in code, I needed a bit of a different approach than what I'm used to. I started out by downloading the executable into a directory on my machine and then proceeded to try to invoke commands against the cli with a sample audio file. I wanted to see if I could effect the file reduction, but was also hoping to listen a bit to the before and after files to validate what I'd read on 'the webs' about the imperceptible-ness of the loss of quality. Utilizing their docs, in a relatively short amount of time, I was able to get what I wanted to happen from the cli, which looked roughly as follows: `ffmpeg.exe -y -i \"{TempLocalFilePath}\" -codec:a libmp3lame -b:a {ReducedBitRate} -ac 1 \"{outputFilePath}\"`. That was such a great feeling. Step 1 complete! Still had a ways to go though.

### Taking one step at a time
The next step was to get the cli invocation working from C#. For that, I created a small console program, utilizing my already downloaded executable. This was new to me. I'm used to writing programs that do things, not writing programs that utilize other programs to do things. It struck me as a great way forward though. We utilize libraries all the time to perform a variety of actions without the need to write it all ourselves. Why not extend this further? Why not take specialized programs, that have been written in any language at all, and utilize them for their specialty without any understanding of their inner workings. This also turned out to be surprisingly easy:
{% highlight csharp linenos=table %}
var workingDirectory = Path.Combine("C:\...\MyProgram\bin", "ffmpeg");
var pathToExecutable = Path.Combine("C:\...\MyProgram\bin\ffmpeg", "ffmpeg.exe");
var outputFileName = $"{Lecture.Title} - {Lecture.Subtitle} ({Lecture.RecordedOn.ToString("MMM d yyyy")}).mp3";
var outputFilePath = Path.Combine(TempFileDirectory, outputFileName);

var info = new ProcessStartInfo
{
    FileName = pathToExecutable,
    WorkingDirectory = workingDirectory,
    Arguments = $"-y -i \"{OriginalFilePath}\" -codec:a libmp3lame -b:a {ReducedBitRate} -ac 1 \"{outputFilePath}\"",

    RedirectStandardInput = false,
    RedirectStandardOutput = true,
    UseShellExecute = false,
    CreateNoWindow = true
};

using var proc = new Process
{
    StartInfo = info
};
proc.Start();
proc.WaitForExit();

File.Delete(OriginalFilePath);
{% endhighlight %}

At this point, my optimism was really picking up. After days of research and tinkering, significant progress was happening. While the above took a few hours, I was able to see/feel continuous, incremental progress. Following the previous steps, I wanted to perform this invocation from an Azure Function app instead of the Console app.

There are a number of different ways that an Azure function can be triggered. For my specific use case, I saw the easiest way to get started with this was to utilize the `BlobTrigger` provided by the SDK. This is basically a message queue. Azure seemingly emits events when something happens within a blob storage container. The `BlobTrigger` attribute sets up a listener for those events and then gets invoked in response to them. When a new audio file was uploaded into a given root path, via the Azure UI, the Azure function would get triggered in response to the Azure blob storage event and the associated method would get invoked. While this definitely took some time digging through Microsoft's docs, it was relatively easy. The above function was encapsulated in an `AudioFileService` and the resulting function code looked like this:
{% highlight csharp linenos=table %}
[FunctionName("OriginalFileListener")]
public static async Task BlobListener([BlobTrigger(StaticData.ContainerName + "/" + StaticData.UploadedFileRootContainerPath + "/{name}", Connection = "")]ICloudBlob blob, string name, ILogger log, ExecutionContext context)
{
    log.LogInformation($"Executing directory: {context.FunctionAppDirectory}");
    log.LogInformation($"Processing Name: {name}, Size: {blob.Properties.Length}b");
    var podcastMetadata = StaticData.OriginalLectureSeriesMetadata;

    var audioFileService = new AudioFileService(blob, context);
    await audioFileService.ProcessFile(podcastMetadata).ConfigureAwait(false);
    log.LogInformation($"Uploaded Name:{Path.GetFileName(audioFileService.OutgoingBlobReference.Name)}, Size: {audioFileService.OutgoingBlobReference.Properties.Length}b");
}
{% endhighlight %}

> Reviewing the code after all these years, its semi-amusing to see some of the naming and other inconsistencies that were a part of the project in the early days. I remind myself though that the initial release happened in around 10 days. I was definitely moving quickly and spending a lot of time naming things wasn't a priority. I believe that most of that has been corrected over the course of time. I also remember that this project initially felt like I was building a podcast. I even did research into the xml format required for a podcast as that was a desired feature initially. While I have the code to make that happen(not fully tested yet), it hasn't been used to date. It was a good reminder for me though to try to name things for what they are/represent and not for how you think they might be utilized as a part of an application.

### The rough waters
Reviewing the code above again also allowed me to remember that it wasn't all smooth sailing. Getting the above to work locally went relatively smoothly. At this point, I utilized the Azure DevOps UI to create a build/deploy pipeline(utilizing their 'Classic' UI which was all drag and drop) and was able to integrate that into the GitHub repo. I created my Azure function via the Azure UI as well and had the pipeline deploy any code changes out to the function. This went relatively smoothly. My experience with Azure DevOps has been that they are pretty well documented, but it isn't always easy to find the documentation and there are quite a few variables that relate to the built-in paths on the build agent that are very similar but not quite the same. It took quite a number of failed builds and digging through logs to get everything working as expected. Still, it was a relatively easy win.

That is when the next set of difficulties set in though. While everything was working well with my function locally, there were a few things that still needed to be worked out to get it working in Azure. For starters, GitHub doesn't allow you to upload an exe as a part of your repository. That meant that the ffmpeg executable that I'd downloaded locally needed to get into my build artifacts somehow. Luckily, someone else had already created a NuGet package to do just that. I deleted the .exe I had downloaded, installed the `FFmpeg.Win64.Static` NuGet package and was past hurdle #1.

The next hurdle turned out to be much more difficult to track down and to solve. When executing code via an Azure function, it is obviously being run on a computer in Azure's cloud. Since that is the case, Azure, understandably, has some security measures in place to protect the execution of any given function code from effecting the machine its running on and the other code that is also running on that machine. These measures are intended to shield the execution from constraining resources for other code running on the same machine as well as protecting the code from any sort of cross contamination. One function app shouldn't be able to have an effect on any other function app, including its code files, etc.

That being the case, via multiple failed attempts and copious amounts of logging, I discovered that there were two main hurdles to work around. Invoking the .exe as I had done above worked great locally, but the Azure function only allowed execution like that to be performed from within a specific directory. Apparently that directory is the one given the permissions, by the executing context, to perform such actions. Presumably it is a well known location and extra locked-down to allow this capability. The other location to pin down was one in which file manipulation was allowed. The function needed to read(exe had to read it in order to change the bitrate) and write(download the original file from blob storage and write the reduced file post-manipulation) files to the directory in order to function correctly.

The location required for the file manipulation turned out to have a relatively straightforward solution. There are many things I like and find intuitive about .NET, but the APIs around the file system have always been tricky for me. Combine that with the fact that the Azure function is working in a virtual file system and the problem felt even tougher. `Directory` has a method <a href="https://learn.microsoft.com/en-us/dotnet/api/system.environment.getfolderpath?redirectedfrom=MSDN&view=net-7.0#System_Environment_GetFolderPath_System_Environment_SpecialFolder_">`GetFolderPath`</a> which takes in an <a href="https://learn.microsoft.com/en-us/dotnet/api/system.environment.specialfolder?view=net-7.0">`Enum`</a> that enables the retrieval of 'special' folders in Windows. Scrolling through the list in Microsoft's docs, I didn't see any that seemed to point to the 'temporary' file location. After a quick search I came up with `Path.GetTempPath()`. On a Windows machine, this points to `C:\Users\{UserName}\AppData\Local\Temp`. This is a folder used by many programs to temporarily place files needed for a specific, usually, short-lived action. I didn't know how it would work in an Azure function, but thankfully it 'just' worked.

The other issue I needed to figure out was how I could get the path to the exe and execute it. While the Microsoft docs are pretty good, most of the examples show the most basic function/trigger and don't provide too much more. Anything more advanced than that usually takes quite a bit of Bingling and investigation. For this, I needed to look at the WebJobs SDK documentation to discover the <a href="https://learn.microsoft.com/en-us/azure/app-service/webjobs-sdk-how-to#executioncontext">`ExecutionContext`</a>. The `ExecutionContext` has two properties of interest: `FunctionDirectory`, `FunctionAppDirectory`. Through a little trial and error, I discovered that the `FunctionAppDirectory` was the one that worked for me. While not hard to discover, the fact that I had to push/build/deploy multiple times throughout the investigation process, made the work tedious.

While the file manipulation piece of the code was definitely the most time intensive portion of the original development, it was also the most rewarding. I was able to time-box the effort, push my creativity and development chops and walk away from it with something that I was quite proud of. I was able to meet my timeline and not compromise on what I felt was the 'right way' to building the application. In a world where software development is often prodded to skip steps and compromise on software quality for a business' bottom line, it was invigorating to instead put the stewardship of the web and the end user first.

### Gained focus
Through this initial micro-implementation, I was able to focus on one aspect of the broader problem to solve. In doing so I had to:
1. Understand what optimizations could be made to an audio file and which of those optimizations would effect a lower file size without impacting perceptible quality.
2. Determine how to reduce the bitrate of an audio file.
3. Learn how to execute an executable via C#.
4. Understand the directory structure/permissions for an Azure function.

### Reduced distractions
In the previous micro-project, I was able to reduce distractions from the outset by breaking the entire application I wanted to develop into 3 areas of focus. As I progressed through this micro-project, I reduced distractions by determining what was the next incremental piece of code that would get me closer to the over all target. By taking the project in small increments, I was able to remove big distractions via focusing on little problems every step of the way. This reduced the unknown 'big problem' that would've compounded if the whole process was tackled at one go. Knowing each step of the way that 90% of that particular problem was solved made solving the remaining 10% seem achievable.
1. I was able to research to know what problem I needed to solve to reduce the file size.
2. Identify a program that was able to perform the reduction.
3. Iterate quickly via the cli on exactly which parameters were needed to perform the reduction.
4. Focus on how to invoke an executable with C#.
5. Determine how to get an executable into the release package since source code's limitations wouldn't allow it there.
6. Pinpoint how to run the entire piece of code via a deployed Azure function.

The difficult and most unknown portion of the initial deliverable was now out of the way. In addition to this, I added two small endpoints, one to return the audio file and one to return information about the lecture. The next step was to create the frontend so that users could actually access these files.
