---
layout: post
permalink: /fullstack/file-download-options
title: ASP.NET File Download Options
excerpt: ''
---

I work primarily as a web developer utilizing ASP.NET as the backend for my applications. In many web applications, there is a need to provide the user with a file from the server. On StackOverflow I've encountered the same type of question many times, "How can I allow the user to click a link and download a file? I'm utilizing {Insert JS framework hotness of the day here} on the front end, supported by ASP.NET on the backend.

    namespace Application.Controllers
    {
    [RoutePrefix("api/examples")]
    public class ExamplesController : ApiController
    {
        [Route("")]
        //public async Task<HttpResponseMessage> Get()//Task<IHttpActionResult> Get()
        public async Task<IHttpActionResult> Get()//Task<IHttpActionResult> Get()
        {
            await Task.Factory.StartNew(() => 1);
            //return new FileResult();
            //var id = Guid.NewGuid().ToString();
            //HttpContext.Current.Session. = new HttpSessionStateBase();
            //HttpContext.Current.Session.Add(id, "ProviderPortal.docx");
            var id = HttpUtility.UrlPathEncode("ProviderPortal.docx");
            //var response = Request.CreateResponse(HttpStatusCode.Moved);
            //var a = Request.RequestUri.GetLeftPart(UriPartial.Authority);
            //var b = Path.Combine(a, id);
            //response.Headers.Location = new Uri(Request.RequestUri + $"/{id}");
            //return response;
            return Ok(new Uri(Request.RequestUri + $"/{id}"));
        }

        [Route("{id}")]
        public async Task<IHttpActionResult> Get(string id)
        {
            //var a = HttpContext.Current.Session[id];
            await Task.Factory.StartNew(() => 1);
            return new FileResult();
        }
    }
    }

public class FileResult : IHttpActionResult
{
    public static string FilePath = Path.Combine(HttpContext.Current.Server.MapPath("~/Temp"), "foo.docx");

    public Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        var MemoryStream = new MemoryStream();
        using (var file = File.OpenRead(FilePath))
        {
            byte[] bytes = new byte[file.Length];
            file.Read(bytes, 0, (int)file.Length);
            MemoryStream.Write(bytes, 0, (int)file.Length);
        }

        MemoryStream.Position = 0;

        var response = new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StreamContent(MemoryStream)
        };

        var contentType = MimeMapping.GetMimeMapping(Path.GetExtension(FilePath));
        response.Content.Headers.ContentType = new MediaTypeHeaderValue(contentType);
        response.Content.Headers.ContentDisposition = new ContentDispositionHeaderValue("attachment");
        response.Content.Headers.ContentDisposition.FileName = "foo.docx";

        return Task.FromResult(response);
    }
    }


    import {HttpClient, json} from 'aurelia-fetch-client';
    import {autoinject} from 'aurelia-framework';

    @autoinject
    export class ExamplesService {
    public constructor(private http: HttpClient) {
        http.configure(config => {
            config.useStandardConfiguration();
        });
    }
    public getFile(): void {
//        return this.http
               this.http
                   .fetch(`api/examples`,
                    {
                        headers: {'content-type': 'application/json'},
                        method: 'GET'
                    })
                    .then(response => response.text())
                    .then(url => {
                        url = url.replace('"', '');
                        /* tslint:disable-next-line */
                        console.log(url);
                        window.location.assign(url);
                    });
                    // .then(resp => {
                    //     // let a = resp.headers.get('Content-Disposition').split(';').find(val => val.startsWith(' filename'));
                    //     // /* tslint:disable-next-line */
                    //     // console.log(a);
                    //     return resp.blob();
                    // })
                    // .then(blb => {
                    //     // new File(blb, 'foo');
                    //     (<any> blb).lastModifiedDate = new Date();
                    //     (<any> blb).name = 'foo';
                    //     let a = URL.createObjectURL(blb);
                    //     /* tslint:disable-next-line */
                    //     console.log(a);
                    //     // a.anchor.;
                    //     return URL.createObjectURL(blb);
                    // });
    }
}

import {ExamplesService} from '../services/examples';
import {BindingEngine, autoinject} from 'aurelia-framework';

@autoinject
export class BusinessComponentsList {
    public download() {
        this.examplesService.getFile();
        // .then(url => {
        //     /* tslint:disable-next-line */
        //     console.log(url);
        //     url += 'foo';
        //     window.location.assign(url);
        // }).catch(err => {
        //                 /*tslint:disable-next-line */
        //                 console.log(err);
        //             });

    }
}`


tutaurelia.net/2016/01/10/export-data-to-excel-from-an-aurelia-web-application#crayon-5875284a19527589411893
