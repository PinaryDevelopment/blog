---
layout: post
permalink: /c-sharp/entity-framework/testing-a-view-with-sqlite
title: Testing a View with Entity Framework Core and SQLite
excerpt: An approach to unit testing a view utilizing SQLite and Entity Framework Core.
categories: ['C#', 'Entity Framework Core', 'SQLite', 'NUnit', 'unit testing', 'automated testing']
---

I encountered an interesting problem/solution the other day that I thought would be fun to write a short post about.

I'm working with an organization at the moment that utilizes Entity Framework Core as its ORM. This organization is just getting started with unit testing and decided to use NUnit as their testing framework. The situation we ran into isn't NUnit specific, but I mention it as the code samples have NUnit specific attributes. The situation was as follows:

> Based on information this organization has collected on its users, it auto-generates some aggregated information on a user's interests and populates a table with that information. This auto-generation process is owned by one team and exists in a database that isn't used solely by this team's application and therefore isn't accessed directly with Entity Framework.

> In an effort to enhance user experiences, they wanted to provide an interface for the users to view those auto-generated interests and remove them or add new ones. The approach that was taken for this problem was to utilize two tables and a view. One table(the one external to this project) contained the auto-generated interests. The other contained all of the user interests' change requests. The view then combined the two, utilizing the user's table to add to/mask from the auto-generated table. This allowed the organization to preserve the additional data it generated, related to those terms, in case the user wanted to add the term back or to help enhance the auto-generation mechanism's accuracy in the future.

For instance:

**Auto-generated table**

| UserId  | Interest    | Score  |
| :-----: | ----------- | :----: |
| 1       | restaurants | 5      |
| 1       | games       | 15     |


**User interests table**

| UserId  | Interest    | Action |
| :-----: | ----------- | ------ |
| 1       | restaurants | delete |
| 1       | nature      | add    |


**Combined interests view**

| UserId  | Interest    |
| :-----: | ----------- |
| 1       | games       |
| 1       | nature      |

As this was an area where a bug was recently discovered, it seemed a ripe area to add unit tests. The official [Entity Framework Core documentation](https://docs.microsoft.com/en-us/ef/core/providers/in-memory/) seems to favor utilizing SQLite slightly for testing, so the team decided to take that approach.

The `DbContext` for the types used looked as follows:

{% highlight csharp linenos=table %}
namespace PeinearyDevelopment.BusinessComponents.DataAccess
{
    public class UserDbContext : DbContext
    {
        internal DbSet<UserInterestEditDto> UserInterestEdits { get; set; }

        public DbSet<UserInterestDto> UserInterests
        {
            get { return Set<UserInterestDto>(); }
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<UserInterestEditDto>(entity =>
            {
                entity.ToTable("UserInterestEdits", "dbo");
            });

            modelBuilder.Entity<UserInterestDto>(entity =>
            {
                entity.ToTable("V_UserInterests", "dto");
                entity.HasKey(interest => new { interest.UserId, interest.Interest });
            });
        }
    }
}
{% endhighlight %}

As can be seen, the `UserInterestEdits` aren't ever exposed directly, so that `DbSet` is made to be `internal` and since the view: `UserInterests` is `readonly`, it only implements the `get` method for it's `DbSet`. Also of note, while slightly confusing, the `DbSet` is linked to its view with the `ToTable` method as a view is treated like a virtual table.

Normally it is considered best practice to write a failing test first and then update the code to make the test succeed. For the brevity of the article and as the bug isn't really relevant to this article, I'm writing things a bit out of order. Once our bugfix was in the place, the code under test is as follows:

{% highlight csharp linenos=table %}
namespace PeinearyDevelopment.BusinessComponents.DataAccess
{
    public enum Action
    {
        Add = 0,
        Delete = 1
    }

    public class UsersDal : IUserModifierDal
    {
        private UserDbContext DbContext { get; }

        public UsersDal(UserDbContext dbContext)
        {
            DbContext = dbContext;
        }

        public async Task MergeInterests(int userId, string[] interests, Action action)
        {
            var hasChanges = false;
            foreach (var interest in interests)
            {
                var dbInterest = DbContext.UserInterestEdits.FirstOrDefault(edit => edit.UserId == userId && edit.Interest == interest);

                if ((dbInterest != null && action == Action.Delete) || (action == Action.Add && dbInterest.Action == Action.Delete))
                {
                    DbContext.UserInterestEdits.Remove(dbInterest);
                    hasChanges = true;
                }
                else if (dbInterest == null)
                {
                    var newUserInterestEdits = DbContext.Set<UserInterestEditDto>()
                                                        .Add(new UserInterestEditDto
                                                              {
                                                                Action = action,
                                                                UserId = userId,
                                                                Interest = interest
                                                              });
                    hasChanges = true;
                }
            }

            if (hasChanges)
            {
                await DbContext.SaveChangesAsync().ConfigureAwait(false);
            }
        }

        public Task<UserInterestDto[]> GetUserInterests(int userId)
        {
            return DbContext.UserInterests
                            .AsNoTracking()
                            .Where(e => e.UserId == userId)
                            .ToArrayAsync();
        }
    }
}
{% endhighlight %}

Since we are using dependency injection to provide instances of interface implementations to our classes through their constructors, we need to setup the DI container for our tests as well. That looks as follows:

{% highlight csharp linenos=table %}
namespace PeinearyDevelopment.BusinessComponents.DataAccess.UnitTests
{
    [SetUpFixture]
    public class Setup
    {
        private static ServiceProvider _serviceProvider;

        public static ServiceProvider ServiceProvider
        {
            get
            {
                if (_serviceProvider == null)
                {
                    var configurationContainer = new ServiceCollection();

                    configurationContainer.AddDbContext<UserDbContext>(options => options.UseSqlite("DataSource=:memory:"));
                    configurationContainer.AddScoped<IUserModifierDal, DataAccess.UsersDal>();

                    _serviceProvider = configurationContainer.BuildServiceProvider();
                }

                return _serviceProvider;
            }
        }
    }
}
{% endhighlight %}

___
> **NOTE**: [Entity framework's documentation](https://docs.microsoft.com/en-us/ef/core/providers/sqlite/limitations) seems to still state that schema support is a limitation for the SQLite provider. While I have experienced this in the past and had to use a workaround, currently, the SQLite provider seems to handle schemas gracefully and I didn't find the need for working around this limitation

**WORKAROUND**:
{% highlight csharp linenos=table %}
public UserDbContext(DbContextOptions<UserDbContext> options) : base(options)
{
    IsTestRun = options.FindExtension<SqliteOptionsExtension>() != null;
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<UserInterestDto>(entity =>
    {
        if (IsTestRun)
        {
            entity.ToTable("dto_V_UserInterests");
        }
        else
        {
            entity.ToTable("V_UserInterests", "dto");
        }
    }
}
{% endhighlight %}
___

Now for the tests. As the first few were going to be reading/writing to the actual table on our context, we anticipated they would be easy.

{% highlight csharp linenos=table %}
namespace PeinearyDevelopment.BusinessComponents.DataAccess.UnitTests.UsersDal
{
    [TestFixture]
    public class MergeInterests
    {
        private IUserModifierDal UsersDal { get; }

        public MergeInterests()
        {
            var serviceProvider = Startup.GetServiceProvider();
            UsersDal = serviceProvider.GetService<IUserModifierDal>();
        }

        [Test]
        [Description(@"Given:
                        A user
                        When:
                        A new user provided interest is added
                        Then:
                        The interest should be returned for that user")]
        public async Task AddUserProvidedInterestToUser()
        {
            /* arrange */
            const int userId = 1;
            const string userProvidedInterest = "nature";

            /* act */
            await UsersDal.MergeInterests(userId, new[] { userProvidedInterest }, Action.Add).ConfigureAwait(false);

            /* assert */
            var interests = await UsersDal.GetUserInterests(userId).ConfigureAwait(false);

            Assert.IsTrue(interests.Any(interest => string.Equals(interest.Interest, userProvidedInterest, System.StringComparison.Ordinal)));
        }

        [Test]
        [Description(@"Given:
                        A user with a user provided interest
                        When:
                        The user provided interest is deleted
                        Then:
                        The interest should not be returned for that user")]
        public async Task RemoveUserProvidedInterestFromUser()
        {
            /* arrange */
            const int userId = 1;
            const string userProvidedInterest = "nature";
            await UsersDal.MergeInterests(userId, new[] { userProvidedInterest }, Action.Add).ConfigureAwait(false);

            /* act */
            await UsersDal.MergeInterests(userId, new[] { userProvidedInterest }, Action.Delete).ConfigureAwait(false);

            /* assert */
            var interests = await UsersDal.GetUserInterests(userId).ConfigureAwait(false);

            Assert.IsFalse(interests.Any(interest => string.Equals(interest.Interest, userProvidedInterest, System.StringComparison.Ordinal)));
        }
    }
}
{% endhighlight %}

Those seemed reasonable, so we decided to run the tests...and they both failed. We were scratching our heads for a few moments as to why because we stepped through the code and saw that the `UserInterestEdits` table was getting updated as expected, but our assertions were still failing.

The reason was quite simple actually, but it took us a minute to figure out what was going on. Entity Framework was creating both DbSets for us, so in stepping through the code all seemed well. The obvious problem though is that while in SQL land our `UserInterests` are coming from a view, in SQLite land, there was no view. Entity Framework created a table for `UserInterests` to execute queries against, but it wasn't reflecting the additions to the `UserInterestEdits` table. Once we realized that, it took a couple of iterations, but we came up with the following addition to our `Setup` class:

{% highlight csharp linenos=table %}
[OneTimeSetUp]
public async Task InitializeDatabase()
{
    var context = ServiceProvider.GetService<UserDbContext>();
    await context.Database.OpenConnectionAsync().ConfigureAwait(false);
    context.Database.ExecuteSqlCommand(createView);
    context.SaveChanges();
}

/*
    * Entity Framework sets up tables for all of its entities in SQLite
    * This entity set is pulling from a View.
    * In order to emulate that, we need to DROP the auto-generated table and CREATE the VIEW instead.
*/
private readonly string createView = $@"
DROP TABLE V_UserInterests;

CREATE VIEW V_UserInterests AS
SELECT
    UserId,
    Interest
FROM UserInterestEdits
WHERE ActionType = {(short)Action.Add};";
{% endhighlight %}

We now ran the tests again and *SUCCESS*, they passed!

The only problem we had now was that the bugfix was actually in the area where the user was manipulating the auto-generated interests. We set about writing the tests for those and quickly realized an issue. The other table that is used to create the view was in another database and wasn't actually a part of our Entity Framework DbContext. Given the previous work we had done though, a path forward became clear pretty quickly.

First we modified the `createView` string in our `Setup` class to:

{% highlight csharp linenos=table %}
private readonly string createView = $@"
DROP TABLE V_UserInterests;

CREATE TABLE AutoGeneratedUserInterests (
    UserId INT PRIMARY KEY NOT NULL,
    Interest NVARCHAR NOT NULL,
    Score NUMERIC(7, 4) NOT NULL
);

CREATE VIEW V_UserInterests AS
SELECT
    UserId,
    Interest
FROM UserInterestEdits
WHERE ActionType = {(short)Action.Add}

UNION

SELECT
    auto.UserId,
    auto.Interest
FROM AutoGeneratedUserInterests auto
LEFT OUTER JOIN UserInterestEdits edits ON auto.UserId = edits.UserId AND auto.Interest = edits.Interest
WHERE edits.UserId IS NOT NULL
      AND edits.ActionType <> {(short)Action.Delete};";
{% endhighlight %}

This creates a 'mocked' auto generated interests table and creates the view as a union between the two tables, masking any auto-generated interests the user wanted to remove. We were then able to create our remaining tests.

{% highlight csharp linenos=table %}
[Test]
[Description(@"Given:
                A user with a system generated interest
                When:
                The system generated interest is deleted
                Then:
                The interest should not be returned for that user")]
public async Task RemoveSystemGeneratedInterestFromUser()
{
    /* arrange */
    const int userId = 1;
    const string systemGeneratedInterest = "nature";

    await CreateSystemGeneratedInterest(userId, systemGeneratedInterest).ConfigureAwait(false);

    /* act */
    await UsersDal.MergeInterests(userId, new[] { systemGeneratedInterest }, Action.Delete).ConfigureAwait(false);

    /* assert */
    var interests = await UsersDal.GetUserInterests(userId).ConfigureAwait(false);

    Assert.IsFalse(interests.Any(interest => string.Equals(interest.Interest, systemGeneratedInterest, System.StringComparison.Ordinal)));
}

[Test]
[Description(@"Given:
                A user with a system generated interest that has been deleted
                When:
                The system generated interest is added
                Then:
                The interest should be returned for that user")]
public async Task AddRemovedSystemGeneratedInterestFromUser()
{
    /* arrange */
    var userId = 1;
    const string systemGeneratedInterest = "interest";

    await CreateSystemGeneratedInterest(userId, systemGeneratedInterest).ConfigureAwait(false);

    await UsersDal.MergeInterests(userId, new[] { systemGeneratedInterest }, Action.Delete).ConfigureAwait(false);

    /* act */
    await UsersDal.MergeInterests(userId, new[] { systemGeneratedInterest }, Action.Add).ConfigureAwait(false);

    /* assert */
    var interests = await UsersDal.GetUserInterests(userId).ConfigureAwait(false);

    Assert.IsTrue(interests.Any(interest => string.Equals(interest.Interest, systemGeneratedInterest, System.StringComparison.Ordinal)));
}

private async Task CreateSystemGeneratedInterest(int userId, string systemGeneratedInterest)
{
    var sql = $@"INSERT INTO AutoGeneratedUserInterests
                    (UserId, Interest)
                  VALUES
                    ({userId}, '{systemGeneratedInterest}')";

    await DbContext.Database.ExecuteSqlCommandAsync(sql).ConfigureAwait(false);
    await DbContext.SaveChangesAsync().ConfigureAwait(false);

    var systemGeneratedInterests = await UsersDal.GetUserInterests(userId).ConfigureAwait(false);
    Assert.IsTrue(systemGeneratedInterests.Any(interest => string.Equals(interest.Interest, systemGeneratedInterest, System.StringComparison.Ordinal)));
}
{% endhighlight %}

There is definitely a less than ideal aspect to this approach. The view and external table need to be replicated here and if either of them change, the updates need to be made in this project as well to continue to have the tests be valid. On the other hand, especially as these are unit tests, this approach allows us to mock out the external dependencies and have our tests focus on asserting the validity of their internal logic.
