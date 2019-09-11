---
layout: post
permalink: /c-sharp/curious-encoding-caper
title: The Curious Encoding Caper
excerpt: Fix for an ASP.NET application returning a csv FileResult that was UTF-8 encoded, not displaying French accent symbols properly in Excel.
categories: ['C#']
---


I came across an interesting thing while working on a project the other day. There was a bug that was logged for the product. 

When a customer entered some French into one of our forms(some of the words had special French symbols and accents, i.e.: Contrecœur, sécurité) and submitted the form. The information, as entered, was saved into our database. We could open that record back up in the web application and everything would display as expected. The web application had a button that allows customers to export some data to a csv file and this is where things got interesting. When clicking on the button, the csv file would download and clicking on the file would open it up by default in Excel. The above words were then displayed as: ContrecÅ“ur, sÃ©curitÃ© respectively. An internal member of our team noted that they were able to export the same record to csv from SSMS(Sql Server Management Studio) and it would display correctly in Excel.

Looking into the code, the line in question looked as follows: `return File(new UTF8Encoding().GetBytes(sb.ToString()), "application/octet-stream", "mycsv.csv");`.

Those were the details I had to work with and as all good developers do, I started Bingling. I noticed that if I opened the csv in Notepad++, the characters would display as expected which led me to start thinking there was some issue with Excel. I searched for properly encoding csv's for Excel and found some interesting and seemingly related posts talking about Excel's inability to do this and exporting to csv well. This gave me false hope that I was getting close, but throughout it all, I didn't see anything that seemed to answer the difficulty we were facing.

I changed search tactics a bit and found a bunch of seemingly related Stack Overflow Q&As.

- [#1](https://stackoverflow.com/questions/393647/response-content-type-as-csv)
  - I tried changing the mime-type provided to the `FileResult` to be the proper one for csv files(`text/csv`).
  - I tried utilizing ASCII encoding instead(this was really just fumbling in the dark, but I was trying things to see if I could get any differences to occur to possibly help my future guesswork).
  - I tried updating the `Response.Charset = "utf-8";`.
- [#2](https://stackoverflow.com/questions/2014069/windows-1252-to-utf-8-encoding)
  - Seeing an answer from Jon Skeet got me quite excited.
  - I tried a few different encodings such as `Encoding.GetEncoding(1252);`. Reading more about this online seemed to indicate that this is a windows specific encoding and our users aren't limited to windows users, so I didn't want to take this route.
- [#3](https://stackoverflow.com/questions/3777874/write-to-csv-file-and-export-it)
  - I found this [answer](https://stackoverflow.com/questions/3777874/write-to-csv-file-and-export-it#answer-28806716).
  - This led to a [GitHub project](https://github.com/jitbit/CsvExport).
  - I perused the code and stumbled across [this line](https://github.com/jitbit/CsvExport/blob/master/CsvExport.cs#L202)

After all of the above, I was able to try a few things and came up with the following change:

- before -> `return File(new UTF8Encoding().GetBytes(sb.ToString()), "application/octet-stream", "filename.csv");`
  - after -> `return File(Encoding.UTF8.GetPreamble().Concat(Encoding.UTF8.GetBytes(sb.ToString())).ToArray(), "text/csv", "filename.csv");`


Once all was said and done, the fix seemed pretty straight forward and insignificant. I struggle to understand why I couldn't find any questions/documentation relating to this as once I knew what the fix was, the method in use seems pretty well documented by [Microsoft](https://docs.microsoft.com/en-us/dotnet/api/system.text.encoding.getpreamble?view=netframework-4.8).

Basically, without this Byte Order Mark([BOM](https://en.wikipedia.org/wiki/Byte_order_mark)) on the file, Excel wasn't sure what the proper encoding was for the file and was taking an incorrect guess(what it was guessing, I'm not sure :) ).

FWIW, if you are as curious as I am, the BOM in this instance seems to consist of three bytes: `239 187 191` OR `11101111 10111011 10111111` in binary!
