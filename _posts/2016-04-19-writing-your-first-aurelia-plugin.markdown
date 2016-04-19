---
published: true
title: Writing your first Aurelia plugin
layout: post
tags: [aurelia, aurelia-plugin, plugin]
categories: [aurelia]
---
![Aurelia](http://aurelia.io/images/main-logo.svg)

Have you begun using Aurelia? No? Then it's time to start! The possibilities are endless. But I won't go into detail how and why you should get started with Aurelia. In this article I will go over how to get started writing plugins for aurelia applications.

In my application, I wanted to expose an aurelia custom attribute that would be available across all my templates without having to import the resources via ```<require/>```. 

<b>Enter plugins! </b>

A good place to get started is by looking at aurelias skeleton plugin,  [check it out here]( https://github.com/aurelia/skeleton-plugin )

I will try to explain all the parts of that skeleton project below ( as of 2016-04-19 ). Some files are omitted due to it's obviousness and are not mentioned in this post.
<ul>
<li><b>build/</b> : collection of gulp build scripts to produce output</li>
<li><b>dist/</b>: output produced by build scripts</li>
<li><b>doc/</b>: changelog and javascript documentation</li>
<li><b>src/</b>: sourcesfiles for the plugin </li>
<li><b>test/</b>: karma test files </li>
<li><b>karma.conf.js</b>: Karma configuration file</li>
<li><b>package.json</b>: Package configuration file </li>
</ul>

You can either download all the sourcefiles from the skeleton project or fork it and continue from there. This post will explain how to use your plugin as a jspm package from github.

After you downloaded the sourcesfiles you are going to be working out of the <b>src/</b> folder so I suggest you remove all default files and start from scratch.

So the first step is to give your plugin a name. For this example we will be fake creating a plugin that makes thumbnails, a good name for our plugin would be <b>aurelia-thumbnail-plugin</b> and we create a file called ```aurelia-thumbnail-plugin.js``` inside <b>src/</b>.

So we start out by letting the world know what it is and what it does by modifying our package.json file. I'd suggest you change the name, version <i>(careful, you need to use a value parsable by [semver](https://docs.npmjs.com/misc/semver) here or npm won't parse your package.json)</i>, description, keywords, homepage, bugs, license, author, main <i>( this should point our your newly created ```aurelia-thumbnail-plugin.js``` file)</i>, repository.  

You can read more about package.json [here](https://docs.npmjs.com/files/package.json)

Now we need to write an entry point for our plugin so that aurelia can use it. We need to export the following. 

<ul>
<li>A configure function </li>
<li>Any classes relevant to your plugin </li>
</ul>

Our example:

{% highlight javascript %}

import {Configuration} from './plugin-configuration';

export function configure(aurelia, callback) {
  aurelia.globalResources('./resources/thumbnail-custom-attribute');
  let config = new Configuration(aurelia);
  if (callback !== undefined && typeof(callback) === 'function') {
    callback(config);
    return;
  }
  config.useDefaults();
}
export {Configuration} from './plugin-configuration';

{% endhighlight %}

So first of all, you need to define what resources should be available to your aurelia plugin. Since this plugin will be a custom attribute we register our custom attribute into aurelia by calling ```aurelia.globalResources(resource);```

Now the resource can be any type of aurelia resource, custom element, custom attribute or a template. Anything that would normally go into your ```<require/>``` inside your view.

We also want to let our consumer of the plugin have a chance to give us some information, maybe a default setting value? If a callback is defined we send our config back to the user, if not we set some default values for ourselves. 

To give the user an idea of what settings are available we also export our Configuration class from ```./plugin-configuration.js file```.

Now that we have registered our plugin resource as a global resource, we just need to build something that would work like any normal custom attribute. 

Lets dive into ```./resources/thumbnail-custom-attribute.js``` file. 

{% highlight javascript %}
import {customAttribute, inject, bindable} from 'aurelia-framework';
import {Configuration} from '../plugin-configuration.js';
@inject(Element, Configuration)
@customAttribute('thumbnail')
export class CustomThumbnailAttribute {
    constructor(element, config) {
	this.element = element;
        this.someValue = config.someDefaultValue;
    }

    attached() {
         //code omitted
     }
    detached() {
	 //code omitted	
    }
}
{% endhighlight %}


Ok, so here is our basic custom attribute as they are written for aurelia. I won't go into how to actually make a thumbnail but I will try to explain where you should write your code.

So first of all we need to import some stuff from aurelia and our config so that we can get a hold of all the default settings provided to us. 
We also let aurelia know what our custom attribute is called by decoration our class with a ```@customAttribute()``` tag. What aurelia will now do is whenever a element decorated with our thumbnail tag is rendered, it will give us control of this element and let us do stuff to it. Equally when the element is detached from the DOM we have a chance to set right or just unload resources used. Pretty simple and if you want to learn more about writing custom attributes or elements I'd suggest you go to [Aurelia docs]( http://aurelia.io/docs.html#/aurelia/templating/1.0.0-beta.1.2.2/doc/article/templating-custom-attributes).

It's also time to write your karma tests, but I will not go into any detail about that.

Now that we have working sourcecode it's time to get our plugin up and running inside aurelia.  For this we need to make sure our sourcecode is building. Make sure you have installed [node](https://nodejs.org/en/) and that gulp is installed globally ```npm install gulp -g```.

Run the following command from your plugin root folder:
```gulp build```

Everything should build and some js files be outputted to your <b>dist/</b> folder. 

If no errors occour it's time to push to your repo, the skeleton-plugin has given some easy to use functions that you can use to bump version numbers etc if you want. Do this by calling ```gulp prepare-release```

After everything is synced to github, you need to [create a release](https://help.github.com/articles/creating-releases/) on github to match your newly pushed version. After this is done you can go back to your aurelia application and run ```jspm install github:<user>/<repo-name> ```, for our example it would be ``` jspm install github:novatrox/aurelia-thumbnail-plugin ```. 

If you want to make your work public inside the jspm registry i'd suggest you make a push request to the [jspm registry](https://github.com/jspm/registry). This will make your plugin available through jspm install <plugin name>.

After everything is installed you can then configure aurelia to use your plugin by writing the following code inside your aurelia startup (main.js).

 
{% highlight javascript %}
import { Configuration } from 'aurelia-thumbnail-plugin';
export function configure(aurelia: Aurelia) {
    aurelia.use
        .standardConfiguration()
        .developmentLogging();
    
 aurelia.use.plugin('novatrox/aurelia-thumbnail-plugin', (config: Configuration) => {
        config.useDefaults();
        config.settings.someValue = "duck";
    });
}
{% endhighlight %}

If everthing worked correctly you should see aurelia logging your plugin as loaded during startup:

DEBUG [aurelia] Loading plugin novatrox/aurelia-thumbnail-plugin.

DEBUG [aurelia] Configured plugin novatrox/aurelia-thumbnail-plugin.

Then simply use your custom attribute like any other

```
<img src="someimage.gif" thumbnail>
```

Hopefully this article has given you some insight how to make aurelia plugins. Please comment if you feel something was left out or unclear.
