configuration
keyvault

            var mockConfiguration = new Mock<IConfiguration>();
            var testApiKey = "FakeApiKey";
            mockConfiguration.Setup(c => c["ApiKey"]).Returns(testApiKey);

HttpClient/Factory
            using Moq.Contrib.HttpClient;

            var handler = new Mock<HttpMessageHandler>();
            var mockHttpClientFactory = handler.CreateClientFactory();
            Mock.Get(mockHttpClientFactory)
                .Setup(x => x.CreateClient("api"))
                .Returns(() =>
                {
                    var client = handler.CreateClient();
                    client.BaseAddress = new Uri("https://www.zipcodeapi.com/rest");
                    return client;
                });

            var testApiJsonString = $"{{\"zip_code\":\"{testPostalCode}\",\"lat\":42.462074,\"lng\":-83.230144,\"city\":\"Southfield\",\"state\":\"MI\",\"timezone\":{{\"timezone_identifier\":\"America\\/ Detroit\",\"timezone_abbr\":\"EDT\",\"utc_offset_sec\":-14400,\"is_dst\":\"T\"}},\"acceptable_city_names\":[],\"area_codes\":[248,947]}}";
            var testUrl = $"https://www.zipcodeapi.com/rest/{testApiKey}/info.json/{testPostalCode}";
            handler.SetupRequest(HttpMethod.Get, testUrl).ReturnsResponse(testApiJsonString, "application/json");
            handler.VerifyRequest(testUrl, Times.Once());

EF InMemory


            var options = new DbContextOptionsBuilder<EntitiesDbContext>()
                                .UseInMemoryDatabase(databaseName: ParseMethodName(MethodBase.GetCurrentMethod()))
                                //.UseModel()
                                .Options;

            var testLocation = new Location
            {
                City = "TestCity",
                Created = new DateTimeOffset(2019, 9, 5, 1, 2, 3, TimeSpan.Zero),
                Latitude = 5,
                Longitude = -6,
                PostalCode = "TestPostalCode",
                State = "TestState"
            };

            using (var context = new EntitiesDbContext(options))
            {
                context.Locations.Add(testLocation);
                context.SaveChanges();

                var controller = new LocationsController(context, mockHttpClientFactory.Object, mockConfiguration.Object);
            }

        private static readonly Regex MethodNameMatcher = new Regex("<(.*?)>");

        private string ParseMethodName(MethodBase methodBase)
        {
            return methodBase.Name != "MoveNext" ? methodBase.Name : MethodNameMatcher.Match(methodBase.DeclaringType.Name).Groups[1].Value;
        }


https://giphy.com/gifs/vvLWidwZNYH5e
