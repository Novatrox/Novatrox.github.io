---
published: true
title: Using a custom model binder in ASP.NET Core to parse JSON in GET request
layout: post
comments: true

#linenos
---
I needed to include the data from a Javascript object in the request to a ASP.NET Core MVC controller, but wanted to use GET as the method. This left me with sending the data on the querystring, however, ASP.NET Core (currently) only parses JSON data from the request body, so I needed something to get the client side data into my server side (request) model.

After searching the web and not finding what I needed, I whipped up a solution based on a custom ModelBinder, which does a naive check if the querystring value is JSON, and then uses JsonConvert to deserialize into the supplied model type.

First we need to implement the actual ModelBinder:

{% highlight csharp %}

using System;
using System.Threading.Tasks;
using Microsoft.AspNet.Mvc.ModelBinding;

public class JsonQueryStringModelBinder : IModelBinder {

	public Task<ModelBindingResult> BindModelAsync(ModelBindingContext bindingContext) {

		ValueProviderResult vpr = bindingContext.ValueProvider.GetValue(bindingContext.ModelName);
		var possibleJson = vpr.FirstValue as string;

		if(!string.IsNullOrEmpty(possibleJson)) {

			if((possibleJson.StartsWith("[{") && bindingContext.ModelType.IsArray) || 
			  possibleJson.StartsWith("{")) {

				try {
					return ModelBindingResult.SuccessAsync(
						bindingContext.ModelName,
						Newtonsoft.Json.JsonConvert.DeserializeObject(
							possibleJson,
							bindingContext.ModelType
						)
					);

				} catch(Exception) {
					// Something wrong with the value, handle or ignore
				}
			}
		}

		return ModelBindingResult.NoResultAsync;
	}
}
{% endhighlight %}

Then we need to register it with in the MVC ModelBinders collection, in the Startup.ConfigureServices method:

{% highlight csharp %}
services.AddMvc().AddMvcOptions(options => {

	// ... your configuration

	int i = 0;
	for(; i < options.ModelBinders.Count; i++) {
		if(options.ModelBinders[i].GetType() == typeof(GenericModelBinder)) {
			break;
		}
	}

	options.ModelBinders.Insert(i, new JsonQueryStringModelBinder());
});
{% endhighlight %}

The reason for inserting the Json Querystring modelbinder before the generic model binder is that otherwise it will process any array properties before we have a chance to try to bind to them.

With the setup out of the way, we can just use any POCO in a request model, and deserialize JSON from the querystring into the object.


