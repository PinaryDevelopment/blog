getting started with indexedDb in Angular app

tried putting it into service and couldn't find the proper order of invocations to get db started

create object store can only be done in onupgradeneeded, but it isn't called ever time

onsuccess made sense, but get error that it can only be done in onupgradeneeded

also wanted to seed the object store with data

solution, needed to seed data out of execution context(based on some user action later)

onupgradeneeded is called whenever the version number is updated...this is provided by you in `window.indexedDB.open('PdCubbyDb', 1);`, not dependent on version of indexedDb in browser(what i thought)


see cubby project for example
