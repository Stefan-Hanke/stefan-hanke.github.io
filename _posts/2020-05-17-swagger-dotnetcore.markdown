---
layout: post
title:  "Swagger on dotnet Core"
date:   2020-05-17
categories: dnc swagger
---

In this post, I'll document my experience while creating a demo swagger server, using aspt.net core. I'm following along [this tutorial](https://docs.microsoft.com/de-de/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-3.1&tabs=visual-studio).

---
---

My first stop was to lookup swagger on GitHub, leading to [swagger-api](https://github.com/swagger-api) project. Then - wait WAT? 100% Java code according to GH? Yeah, it looks like swagger is a Java project, and there are ports for different languages and frameworks. The dotnet Core port is called [SwashBuckle](https://github.com/domaindrivendev/Swashbuckle).

However, SwashBuckle seems to be unmaintained - practically no progress in the last two years (last checking on Feb 21 2018), the [call for maintainers](https://github.com/slatedocs/slate/issues/1127) did not help either. The [documentation](https://github.com/RicoSuter/NSwag
https://docs.microsoft.com/de-de/aspnet/core/tutorials/getting-started-with-nswag?view=aspnetcore-3.1&tabs=visual-studio
) hints at another project called [NSwag](https://github.com/RicoSuter/NSwag), that very much looks maintained.

## First wounds

Followed the tutorial to the point where the app should work, however:


D:\git\github\stefan.hanke\swagger-demo>dotnet run
...
 ---> (Inner Exception #1) System.InvalidOperationException: Error while validating the service descriptor 'ServiceType: Microsoft.Extensions.ApiDescriptions.IDocumentProvider Lifetime: Singleton ImplementationType: Microsoft.Extensions.ApiDescriptions.DocumentProvider': Unable to resolve service for type 'Microsoft.AspNetCore.Mvc.ApiExplorer.IApiDescriptionGroupCollectionProvider' while attempting to activate 'Swashbuckle.AspNetCore.SwaggerGen.SwaggerGenerator'.
 ---> System.InvalidOperationException: Unable to resolve service for type 'Microsoft.AspNetCore.Mvc.ApiExplorer.IApiDescriptionGroupCollectionProvider' while attempting to activate 'Swashbuckle.AspNetCore.SwaggerGen.SwaggerGenerator'.
...

After several hops, in which I obviously did **not** create a webapi, it was working now. It's kind-of confusing, creating an **mvc** type app does not work, at least not out of the box. Whyever.

## The Swagger Document

This is the document created by SwashBuckle, in the demo available [here](https://localhost:5001/swagger/v1/swagger.json). This document is subsequently used by SwashBuckle UI to actually create the frontend that is available [there](https://localhost:5001/swagger/index.html).

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "My API",
    "version": "v1"
  },
  "paths": {
    "/WeatherForecast": {
      "get": {
        "tags": [
          "WeatherForecast"
        ],
        "responses": {
          "200": {
            "description": "Success",
            "content": {
              "text/plain": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/WeatherForecast"
                  }
                }
              },
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/WeatherForecast"
                  }
                }
              },
              "text/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/WeatherForecast"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "WeatherForecast": {
        "type": "object",
        "properties": {
          "date": {
            "type": "string",
            "format": "date-time"
          },
          "temperatureC": {
            "type": "integer",
            "format": "int32"
          },
          "temperatureF": {
            "type": "integer",
            "format": "int32",
            "readOnly": true
          },
          "summary": {
            "type": "string",
            "nullable": true
          }
        },
        "additionalProperties": false
      }
    }
  }
}
```

## XML Documentation

It was also quite easy to let the C# documentation flow into the swagger document. There are three ingredients:

- Activate creating the docs xml file via `<GenerateDocumentationFile>true</GenerateDocumentationFile>` in the csproj file
- Include the xml docs file in swagger in `ConfigureServices()`.
```csharp
var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
c.IncludeXmlComments(xmlPath);
```
- Actually add some documentation.

The swagger document then contains the documentation.

```json
"paths": {
    "/WeatherForecast": {
      "get": {
        "tags": [
          "WeatherForecast"
        ],
        "summary": "Gets all weather forecast reports.",
        "responses": {
```
