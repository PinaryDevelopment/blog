---
layout: post
permalink: /c-sharp/micro-services/base-class-libraries/structure
title: "The Micro-service Base Class Libraries: Structure"
excerpt: Creating a model micro-service to demonstrate core concepts and architecture. Creating the clients.
categories: ['C#', 'architecture', 'template', 'scaffolding', 'SOA', 'Service Oriented Architecture', 'micro-services', 'ASP.NET', 'ASP.NET WebApi 2']
---

---------------------------------------

*As stated in previous posts: I set out to create a model of a micro-service for an organization trying to create a more SOA for their applications. I started off by creating a quick template, but found my code repeating itself and hoped to extract repetitious parts.*

*The projects under discussion in this article are the Api projects.*

*Apis.csproj: This project is the website that will actually expose the service through WebApi2 Controllers. Acting as the BLL, it also contains all of the validation logic for the resource(s) located at its endpoint.*

*Clients.Apis.csproj: This project is a small project that wraps the Api project and allows us(through NuGet) to utilize a strongly typed client for communicating with the component's api.*

---------------------------------------

The business logic layer is generally the essence of an application. Yes, the application needs to look and feel nice for the end-user. The ability to intuitively utilize an application will increase the volume of use it receives and therefore greatly increases its value to an organization. Most applications also need to store some record of its state and be able to retrieve that state record. This needs to be performed quickly and reliably. The business logic layer though is really the glue that holds these two ends together. It aims to ensure the cleanness of the data before it gets stored and refuse it if it isn't correct. It needs to validate the accuracy of the data and the ability of a given request to interact with that data. It is also the conduit through which data should be passed back to the user for further interaction. 

While the interior/exterior design of a building can look quite different from the underlying structure, it must adhere to that structure and utilize it as a starting point from which to build upon. This same idea can be used when discussing the business logic layer of an application. The way users interact with the various components through the UI or the shape of the data while stored might look drastically different than it does in the business logic layer, but they must adhere to the structure imposed upon them by the BLL. 

As with the DAL refactoring, you can see below the drastic changes that were achieved.
### Before
*142 lines of goodness*
{% highlight csharp linenos=table %}
namespace Apis.Controllers
{
    using Config;
    using Contracts;
    using DataAccessContracts;

    using System;
    using System.Collections.Generic;
    using System.Data.Entity.Core;
    using System.Linq;
    using System.Threading.Tasks;
    using System.Web.Http;

    [RoutePrefix("v1/countries")]
    public class CountriesController : BaseApiController
    {
        private ICountriesDal CountriesDal { get; }

        public CountriesController(ICountriesDal countriesDal)
        {
            CountriesDal = countriesDal;
        }

        [HttpGet]
        [Route("{id:int}", Name = "GetCountry")]
        public async Task<IHttpActionResult> Get(int id)
        {
            try
            {
                return Ok(AutoMapperConfig.Mapper.Map<Country>(await CountriesDal.Read(id)));
            }
            catch (ObjectNotFoundException)
            {
                return NotFound();
            }
        }

        [HttpPost]
        [Route("")]
        public async Task<IHttpActionResult> Post(Country country)
        {
            var validationErrors = await ValidateCountry(country);
            if (validationErrors != null && validationErrors.Any()) return BadRequest(string.Join("\n", validationErrors));

            country.CreatedByUserId = IdPrincipal.Id;
            country.CreatedOn = DateTimeOffset.UtcNow;
            country.EffectiveStartDate = country.EffectiveStartDate == DateTimeOffset.MinValue ? DateTimeOffset.UtcNow : country.EffectiveStartDate;
            country.LastUpdatedByUserId = null;
            country.LastUpdatedOn = null;

            var createdCountryDto = await CountriesDal.Create(AutoMapperConfig.Mapper.Map<CountryDto>(country));
            var createdCountry = AutoMapperConfig.Mapper.Map<Country>(createdCountryDto);

            return Created(Url.Link("GetCountry", new { id = createdCountry.Id, controller = "Countries" }), createdCountry);
        }

        [HttpPut]
        [Route("")]
        public async Task<IHttpActionResult> Put(Country country)
        {
            var validationErrors = await ValidateCountry(country);
            if (validationErrors != null && validationErrors.Any()) return BadRequest(string.Join("\n", validationErrors));

            try
            {
                country.LastUpdatedByUserId = IdPrincipal.Id;
                country.LastUpdatedOn = DateTimeOffset.UtcNow;

                var updatedCountryDto = await CountriesDal.Update(AutoMapperConfig.Mapper.Map<CountryDto>(country));
                var updatedCountry = AutoMapperConfig.Mapper.Map<Country>(updatedCountryDto);
                return Ok(updatedCountry);
            }
            catch (ObjectNotFoundException)
            {
                return NotFound();
            }
        }

        [HttpDelete]
        [Route("{id:int}")]
        public async Task<IHttpActionResult> SoftDelete(int id)
        {
            try
            {
                var country = await CountriesDal.Read(id);

                country.LastUpdatedByUserId = IdPrincipal.Id;
                country.LastUpdatedOn = DateTimeOffset.UtcNow;
                country.EffectiveEndDate = DateTimeOffset.UtcNow;

                await CountriesDal.Update(country);
                
                return Ok();
            }
            catch (ObjectNotFoundException)
            {
                return NotFound();
            }
        }

        [HttpDelete]
        [Route("{id:int}/force")]
        public async Task<IHttpActionResult> Delete(int id)
        {
            try
            {
                await CountriesDal.Delete(id);
                return this.NoContent();
            }
            catch (ObjectNotFoundException)
            {
                return NotFound();
            }
        }

        private async Task<List<string>> ValidateCountry(Country country)
        {
            var errorMessages = new List<string>();

            if (string.IsNullOrWhiteSpace(country.FullName)) errorMessages.Add("Country's name can't be blank.");
            if (string.IsNullOrWhiteSpace(country.Alpha2Code)) errorMessages.Add("Country's abbreviation can't be blank.");
            if (country.EffectiveEndDate != null && country.EffectiveEndDate <= country.EffectiveStartDate) errorMessages.Add("Country's effective date range is invalid. EffectiveEndDate must be null or greater than the EffectiveStartDate.");

            var matchingCountries = await CountriesDal.GetMatchingCountries(AutoMapperConfig.Mapper.Map<CountryDto>(country));
            if(matchingCountries.Any())
            {
                if (!string.IsNullOrWhiteSpace(country.FullName) && matchingCountries.Any(c => !string.IsNullOrWhiteSpace(c.FullName) && c.FullName.Equals(country.FullName, StringComparison.OrdinalIgnoreCase)))
                {
                    errorMessages.Add("Country name needs to be unique.");
                }

                if (!string.IsNullOrWhiteSpace(country.Alpha2Code) && matchingCountries.Any(c => !string.IsNullOrWhiteSpace(c.Alpha2Code) && c.Alpha2Code.Equals(country.Alpha2Code, StringComparison.OrdinalIgnoreCase)))
                {
                    errorMessages.Add("Country abbreviation needs to be unique.");
                }
            }

            return errorMessages;
        }
    }
}
{% endhighlight %}

### After
*57 lines to maintain*
{% highlight csharp linenos=table %}
namespace Apis.Controllers
{
    using PeinearyDevelopment.Framework.BaseClassLibraries.Web.Http;

    using Contracts;
    using DataAccessContracts;

    using AutoMapper;
    using System.Threading.Tasks;
    using System.Web.Http;

    [RoutePrefix(Routes.CountryV1BaseRoute)]
    public class CountriesController : DateRangeEffectiveBaseApiControllerBase<Country, CountryDto, int>
    {
        public CountriesController(ICountriesDal dal, IMapper mapper, IContractValidator<Country> contractValidator) : base(dal, mapper, contractValidator)
        {
            CreatedRouteName = Routes.GetCountryNamedRouteName;
            ControllerName = Routes.CountryControllerName;
        }

        [HttpGet]
        [Route("{id:int}", Name = Routes.GetCountryNamedRouteName)]
        public override async Task<IHttpActionResult> Get(int id)
        {
            return await base.Get(id);
        }

        [HttpPost]
        [Route("")]
        public override async Task<IHttpActionResult> Post(Country country)
        {
            return await base.Post(country);
        }

        [HttpPut]
        [Route("")]
        public override async Task<IHttpActionResult> Put(Country country)
        {
            return await base.Put(country);
        }

        [HttpDelete]
        [Route("{id:int}")]
        public override async Task<IHttpActionResult> SoftDelete(int id)
        {
            return await base.SoftDelete(id);
        }

        [HttpDelete]
        [Route("{id:int}/force")]
        public override async Task<IHttpActionResult> Delete(int id)
        {
            return await base.Delete(id);
        }
    }
}
{% endhighlight %}

Again, this was achieved through creating a few [base](https://github.com/PdFramework/BaseClassLibraries/blob/master/Web.Http/IdBaseApiControllerBase.cs) [classes](https://github.com/PdFramework/BaseClassLibraries/blob/master/Web.Http/DateRangeEffectiveBaseApiControllerBase.cs). The controller base classes take care of most of the repetitive work which they are enabled to do due to the assumptions that can be made by the fact that they are accepting base contract classes with a few pre-defined consistent fields. 

*There are a number of 'standard' methods that are exposed through the HTTP protocol: Get, Post, Put, Delete. While there is some debate as to what actions Post and Put map to, I have chosen to implement the Post as a Create method and the Put as an Update method.* 

While Get and Delete generally only have authorization logic around them, Put and Post are where most of the business logic is contained. As can be seen in the ValidateCountry method, there isn't that much that needs to be validated, but if anything tries to invoke the Post or Put methods with a country object that has no name, the ValidateMethod will return a message indicating that the request was invalid. The base controller class can't know anything specifically about the object that is extending it though, so how can it provide proper validation for the object under question? This was achieved through the use of an interface. There is an IValidateContract interface that only defines one method called ValidateContract. Every controller needs to encapsulate an object and that object must have something that implements the method defined on the IValidateContract interface in order for the controller to properly ensure the validity of the data being passed through the controller to the DAL.

This is where all of the business logic resides regarding an object. With this in place, not only have we reduced the amount of redundant code needed to get a micro-service up and running, but we have also encapsulated all of the repetitive code into the base classes and all of the non-repetitive code behind the interface implementation. As I will show in a later post, this also greatly helps us when it comes to testing. There are a lot of tests that don't need to be repeated and each micro-service author can focus on only testing the logic that is specific to the object under creation.

Since the creation of these base classes, I came across [a very interesting post](http://www.strathweb.com/2016/06/inheriting-route-attributes-in-asp-net-web-api/) on StrathWeb which discusses the ability to inherit route attributes in WebApi. This was something I briefly tried to accomplish in the creation of these base classes as I assumed it would further reduce the amount of boilerplate code needed to generate a new api. After finding this post, I haven't had the ability to circle back and see if I could make it work in my base class libraries, but would like to look at it as a future enhancement to this project.

The other aspect of this part of the project was the Clients.Apis project. I really enjoy the simplicity of the HTTP protocol. I like being able to utilize the same technology to connect to the server from a client-side application and to connect from one server to another. I have worked in environments that have relied heavily on WCF services and while they definitely have some advantages, the ability to utilize a light-weight, consistent protocol between applications, be they client-side or server-side has really drawn me to rely on WebApi for my micro-services instead of WCF. 

The one thing that I found myself really missing from WCF though was the experience in .NET of having strongly typed objects to program against. As I found myself utilizing the HttpClient and Newtonsoft.Json libraries to encapsulate this aspect of interacting with the Apis, I decided that it would be nice to have a class that wrapped that functionality, allowing me to create stronly-typed clients for all of our micro-services. This allows us to utilize the benefits of HTTP as well as having strongly-typed objects to program against. The [ApiInvoker](https://github.com/PdFramework/BaseClassLibraries/blob/master/ApiClients/ApiInvoker.cs) creates that small library that makes this all possible. Looking at the [countries client](https://github.com/PdFramework/BaseClassLibraries/blob/master/ApiTestProject/ApiClients/CountriesApiClient.cs), one can see how minimal the code is to provide this nicety. The Clients.Apis and Contracts projects are packaged up into NuGet packages through our build process and deployed to our private NuGet repository, allowing anyone on our team to pull in the desired micro-services through NuGet to incorporate them into their projects.
