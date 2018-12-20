---
layout: post
permalink: /c-sharp/micro-services/base-class-libraries/foundations
title: "The Micro-service Base Class Libraries: Foundations"
excerpt: Creating a model micro-service to demonstrate core concepts and architecture. Data access layer.
categories: ['C#', 'architecture', 'template', 'scaffolding', 'SOA', 'Service Oriented Architecture', 'micro-services', 'Entity Framework']
---

---------------------------------------

*As stated in previous posts: I set out to create a model of a micro-service for an organization trying to create a more SOA for their applications. I started off by creating a quick template, but found my code repeating itself and hoped to extract repetitious parts.*

*The project I hope to discuss in this article is the DataAccess project.*

*DataAccess.csproj: This project contains all of the interactions that should take place between this Api and its associated data-store.*

---------------------------------------

As stated in the previous post, I liken the data access layer of a project to a building's foundation. This layer obviously doesn't function on its own to achieve all of these goals, but if data isn't being persisted or isn't retrievable in a fast and reliable way, then the utility of the program at hand is severely limited. The ability to rely on the data and trace it through its creation and modifications obviously supports the utility of said data and in many instances is critical in the utility of the data. While the trace-ability of the data's state is often times handled through the data access layer, we wanted to create a convention surrounding how trace-ability should be achieved and then bake it into all of our micro-services. I adhered to their preferred approach for this project which entailed, at the moment, only creating a few standard fields around the user that touched the data and when that touch occurred.

With the knowledge of the desired conventions, the base implementation on the data access layer for any given object in their system could mostly be abstracted away, as can be seen in the change that occurred to the data access layer objects before and after the refactoring.

### Before
*81 lines of goodness*
{% highlight csharp linenos=table %}
namespace DataAccess
{
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Data.Entity.Core;
    using System.Linq;
    using System.Threading.Tasks;
    using DataAccessContracts;

    public class CountriesDal : ICountriesDal
    {
        public async Task<CountryDto> Create(CountryDto country)
        {
            using (var context = new CountriesDbContext())
            {
                var createdCountry = context.Countries.Add(country);

                await context.SaveChangesAsync();
                return createdCountry;
            }
        }

        public async Task<CountryDto> Read(int countryId)
        {
            using (var context = new CountriesDbContext())
            {
                var dbCountry = await context.Countries.FirstOrDefaultAsync(country => country.Id == countryId);
                if (dbCountry == null) throw new ObjectNotFoundException();

                return dbCountry;
            }
        }

        public async Task<CountryDto> Update(CountryDto country)
        {
            using (var context = new CountriesDbContext())
            {
                var dbCountry = await context.Countries.FirstOrDefaultAsync(countryDto => countryDto.Id == country.Id);
                if (country == null) throw new ObjectNotFoundException();

                dbCountry.LastUpdatedByUserId = country.LastUpdatedByUserId;
                dbCountry.LastUpdatedOn = DateTimeOffset.UtcNow;
                dbCountry.Alpha2Code = country.Alpha2Code;
                dbCountry.Alpha3Code = country.Alpha3Code;
                dbCountry.FullName = country.FullName;
                dbCountry.EffectiveEndDate = country.EffectiveEndDate;
                dbCountry.EffectiveStartDate = country.EffectiveStartDate;
                dbCountry.Iso3166Code = country.Iso3166Code;
                dbCountry.ShortName = country.ShortName;
                dbCountry.PhoneNumberRegex = country.PhoneNumberRegex;
                dbCountry.PostalCodeRegex = country.PostalCodeRegex;

                await context.SaveChangesAsync();
                return dbCountry;
            }
        }

        public async Task Delete(int countryId)
        {
            using (var context = new CountriesDbContext())
            {
                var dbCountry = await context.Countries.FirstOrDefaultAsync(country => country.Id == countryId);
                if (dbCountry == null) throw new ObjectNotFoundException();

                context.Countries.Remove(dbCountry);

                await context.SaveChangesAsync();
            }
        }

        public async Task<IEnumerable<CountryDto>> GetMatchingCountries(CountryDto country)
        {
            using (var context = new CountriesDbContext())
            {
                return await context.Countries.Where(countryDto => countryDto.FullName == country.FullName && countryDto.Alpha2Code == country.Alpha2Code && countryDto.Id != country.Id).ToArrayAsync();
            }
        }
    }
}
{% endhighlight %}

### After
*28 lines to maintain*
{% highlight csharp linenos=table %}
namespace DataAccess
{
    using PeinearyDevelopment.Framework.BaseClassLibraries.DataAccess;

    using DataAccessContracts;

    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Linq;
    using System.Threading.Tasks;

    public class CountriesDal : DalBase<CountryDto, int>, ICountriesDal
    {

        public CountriesDal(DbContext dbContext) : base(dbContext)
        {
        }

        public async Task<IEnumerable<CountryDto>> GetMatchingCountries(CountryDto country)
        {
            using (var context = new CountriesDbContext())
            {
                return await context.Countries.Where(countryDto => countryDto.FullName == country.FullName && countryDto.Alpha2Code == country.Alpha2Code && countryDto.Id != country.Id).ToArrayAsync();
            }
        }
    }
}
{% endhighlight %}

So, what's going on here? Most of the magic lies in the class entitled [DalBase](https://github.com/PdFramework/BaseClassLibraries/blob/master/DataAccess/DalBase.cs). A common acronym thrown around when talking about persisting data is CRUD -> (C)reate, (R)ead, (U)pdate, (D)elete. At a bare-bones level, these are usually the methods you want exposed one way or another to interact with a system's data. You want to be able to create new records, read/access records that are already there, update certain data points in those records and then delete records when they are no longer of interest/utility to the system. There are a few nuances that commonly pop up in relation to these. When discussing deletion, there are soft deletes(you keep the record, but set some sort of flag indicating that the record is no longer in use) and hard deletes(this entails actually deleting the record from the data store, disallowing it to be retrievable in the future completely). When discussing the read, there are commonly three main types of methods that provide this. Get one record by it's id. Get all of the records for a given data set. Search/Filter records of a dataset and return all instances of data rows that match the desired criteria.

The DalBase as implemented above only implements CRUD with the Read being of the type where you get one row by id and the Delete being a hard delete. As can be seen in later updates around the business logic layer, we will see that the soft delete is exposed there and relies on the DAL's Update method. The read method which filters the data set based off of some criteria set isn't as straight forward to implement in a generic way and was left as a **TODO** item in the future. The retrieve all objects from a given dataset type was omitted as well because I see it as a specialized version of the filter method and include that as part of the **TODO** item mentioned before. As can be seen above as well, the DalBase is left very dumb. Outside of the update(which arguably could be done in the BLL), all of the audit update information pertaining to which user performed which action and when, is relegated to the realm of the BLL.

As mentioned before, the 'filter' method for the dal remains to be implemented(it is actually contained in [another project](https://github.com/PdFramework/Filtering) that I've been working on and hope to write about). Once that is complete and integrated though, the developer at hand should be able to create a new project and utilizing the base dal class create a DAL class for any object with a few lines of code. 

All of this seems great, but what if there is a special scenario that doesn't adhere cleanly to the implementation given above? Do we need to throw everything out and start all over? NO, when creating a base class, it should be known that you can override a method in the class that is extending the base class. If we had some situation in which the default implementation wouldn't cut it, all of these methods could be overridden in favor of an implementation that is specific to this object. The approach encapsulated in the DalBase implementation though should cover 80% of this business' scenarios and allow them to develop and deploy less tightly coupled objects/systems in a rapid fashion and allow them to more rapidly iterate to push their vision of a new architecture forward.
