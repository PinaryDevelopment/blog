had issues before trying to test ef with sqlite, where that was a dependency of the data access project
therefore it would be pulled in also(and packaged) in every application that relied on that library

wanted to not have that tight coupling this time, but proved more challenging than expected

easy path(online examples/documentation) seems to put a lot in the dbcontext
extension methods like `UseSqlServer` and even more so `ForSqlServerIsMemoryOptimized` are often referenced in overridden methods on dbcontext object
