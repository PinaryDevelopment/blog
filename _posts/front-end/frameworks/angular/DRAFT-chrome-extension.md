should do a post on building chrome extensions?

this one can talk about all the issues experience trying to make the popup project utilize angular

tried to wrap the data stuff(local storage/api call) in a service

everything seemed to be working properly(utilizing console.log's in `tap`), but changes weren't seen in ui

solution was to stop using ngzone and manually triggering change detection as seen in canary project
