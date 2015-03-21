
<h5><em>Keep a look out for updated video to show use of new adapter here in the <a href="https://www.youtube.com/channel/UCMCcqbJpyL3LAv3PJeYz2bg">Appcelerator Alloy Video Series</a></em></h5>
----
###Quick Overview
We all have become accustom to using promises to avoid the callback hell so here we have an example of an [ACS adapter](https://github.com/aaronksaunders/ci.alloy.adapter.two) that supports promises using the [$q javascript library](https://github.com/kriskowal/q/blob/v1/README.md).

First you create your Alloy Appcelerator Cloud Services Models following the steps outlined in the [Appcelerator Alloy Model Documentation](http://docs.appcelerator.com/titanium/latest/#!/guide/Alloy_Collection_and_Model_Objects)

When you are done an object looks similar to this; notice how we defined the ACS class and the name of the associated collection to match the class. This is essential because it is how the ACS Sync Adapter cna construct the appropriate REST API call to interact with the service
```javascript,linenums=true
exports.definition = {
    config : {
        "columns" : {},
        "defaults" : {},
        "adapter" : {
            "type" : "acs",
        },
        "settings" : {
            "object_name" : "photos", // <- match the ACS Object
            "object_method" : "Photos"
        }
    },

    extendModel : function(Model) {
        _.extend(Model.prototype, {});
        return Model;
    },

    extendCollection : function(Collection) {
        _.extend(Collection.prototype, {});
        return Collection;
    }
}
```
For Custom Objects, we can support them by providing the name of  the custom object in the configuration setting property and then set the object_method. See example of a custom object called book
```javascript,linenums=true
exports.definition = {
    config : {
    "columns": {},
    "defaults": {},
    "adapter": {
        "type": "acs",
    },
    "settings": {
        "object_name": "book",
        "object_method": "Objects" //<--indicates a Custom ACS object
       }
    },

    extendModel : function(Model) {
        _.extend(Model.prototype, {});
        return Model;
    },

    extendCollection : function(Collection) {
        _.extend(Collection.prototype, {});
        return Collection;
    }
}
```
So now you can query your custom object like this

```javascript,linenums=true
/**
* gets books and returns a promise
*/
function getBooks() {
    var books = Alloy.createCollection('Book');
    return books.fetch();
}

// need a user object to login with
var aUser = Alloy.createModel('User');

// notice the call to the extended function with no success or error
// callbacks, they are handled by the promise structure
aUser.login("testuserone", "password").then(function(_response) {
    // successful login here!!
    Ti.API.info(' Success:Login, with Promise\n ' + JSON.stringify(_response, null, 2));

    // now query for the books and the success will be handled by 
    // the next `then` function below, else it falls thru to the 
    // error 
    return getBooks(); //&lt;-- returns a promise also!
}).then(function(_bookResp) {
    // here we handle the successful book query
    Ti.API.info(' Success:Books, with Promise\n ' + JSON.stringify(_bookResp, null, 2));
}, function(_error) {
    // ANY errors is the promise chain will fall thru to here
    Ti.API.error(' ERROR ' + JSON.stringify(_error));
});
```
This approach is MUCH cleaner that the old callback approach, give it a try... the adapter still support both approaches.

###Additional changes required to get application to function properly


* You will need to include the q javascript library in your project
* You will need to update your `alloy.js` file to support the models and collections returning the promise from the sync adapter, see `line 10` and `line 36` where we return the result from the sync adapter


The new changes to alloy.js

```javascript,linenums=true
Alloy.C = function(name, modelDesc, model) {
	var extendObj = {
		model : model
	};
	var config = ( model ? model.prototype.config : {}) || {};
	var mod;
	if (config.adapter && config.adapter.type) {
		mod = require("alloy/sync/" + config.adapter.type);
		extendObj.sync = function(method, model, opts) {
			return mod.sync(method, model, opts);
		};
	} else
		extendObj.sync = function(method, model) {
			Ti.API.warn("Execution of " + method + "#sync() function on a collection that does not support persistence");
			Ti.API.warn("model: " + JSON.stringify(model.toJSON()));
		};
	var Collection = Backbone.Collection.extend(extendObj);
	Collection.prototype.config = config;
	_.isFunction(modelDesc.extendCollection) && ( Collection = modelDesc.extendCollection(Collection) || Collection);
	mod && _.isFunction(mod.afterCollectionCreate) && mod.afterCollectionCreate(Collection);
	return Collection;
};

Alloy.M = function(name, modelDesc, migrations) {
	var config = (modelDesc || {}).config || {};
	var adapter = config.adapter || {};
	var extendObj = {};
	var extendClass = {};
	var mod;
	if (adapter.type) {
		mod = require("alloy/sync/" + adapter.type);
		extendObj.sync = function(method, model, opts) {
			return mod.sync(method, model, opts);
		};
	} else
		extendObj.sync = function(method, model) {
			Ti.API.warn("Execution of " + method + "#sync() function on a model that does not support persistence");
			Ti.API.warn("model: " + JSON.stringify(model.toJSON()));
		};
	extendObj.defaults = config.defaults;
	migrations && (extendClass.migrations = migrations);
	mod && _.isFunction(mod.beforeModelCreate) && ( config = mod.beforeModelCreate(config, name) || config);
	var Model = Backbone.Model.extend(extendObj, extendClass);
	Model.prototype.config = config;
	_.isFunction(modelDesc.extendModel) && ( Model = modelDesc.extendModel(Model) || Model);
	mod && _.isFunction(mod.afterModelCreate) &amp;&amp; mod.afterModelCreate(Model, name);
	return Model;
};
```
###Related Links
* [Complete Source Code Available here in GitHub Repository](https://github.com/aaronksaunders/ci.alloy.adapter.two)
* [Beginning Appcelerator Titanium Alloy Videos](http://www.clearlyinnovative.com/beginning-appcelerator-titanium-alloy-videos/)

###Latest Video From Series
<iframe width="420" height="315" src="https://www.youtube.com/embed/90EkPnXsdjU" frameborder="0" allowfullscreen="allowfullscreen"></iframe>