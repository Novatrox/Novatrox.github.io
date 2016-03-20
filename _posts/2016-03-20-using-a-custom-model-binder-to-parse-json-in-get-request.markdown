---
published: true
title: Using a custom model binder to parse JSON in GET request
layout: post
---
{% highlight csharp linenos %}

using Microsoft.AspNet.Mvc.ModelBinding;

public class JsonQueryStringModelBinder : IModelBinder {

	public Task<ModelBindingResult> BindModelAsync(ModelBindingContext bindingContext) {

		ValueProviderResult valueProviderResult = bindingContext.ValueProvider.GetValue(bindingContext.ModelName);
		var possibleJson = valueProviderResult.FirstValue as string;

		if(!string.IsNullOrEmpty(possibleJson)) {

			if((possibleJson.StartsWith("[{") && bindingContext.ModelType.IsArray) || possibleJson.StartsWith("{")) {

				try {
					return ModelBindingResult.SuccessAsync(bindingContext.ModelName, Newtonsoft.Json.JsonConvert.DeserializeObject(possibleJson, bindingContext.ModelType));

				} catch(Exception) {
					// Something wrong with the value
				}
			}
		}

		return ModelBindingResult.NoResultAsync;
	}
}

{% endhighlight %}
