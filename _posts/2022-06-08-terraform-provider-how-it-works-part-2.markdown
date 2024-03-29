---
title:  "Terraform Provider: How it works (Part 2)"
date:   2022-06-08 12:00:00 +0300
tags:   terraform provider
excerpt: ""
header:
    teaser: "https://i.pinimg.com/736x/8c/05/03/8c05039f05f823f61fa2f591819b437b.jpg"
    overlay_image: https://i.pinimg.com/736x/8c/05/03/8c05039f05f823f61fa2f591819b437b.jpg
toc: true
---

The second part of the story about Terraform Provider internals. This part will be more focused on different Provider's
mechanisms and objects implementation in Go.

# Generic Provider structure

Terraform has created [terraform-provider-hashicups](https://github.com/hashicorp/terraform-provider-hashicups) repo where
some example Provider is defined. There is a minimal codebase required for having a working Provider.

Below I want to run through the Terraform Providers' (for both AWS and Hashicups) source files and explain the main parts required for the Provider to work.

Let's go through the files in the repo and try to understand what is going on here.

The Provider's entry point is defined within a `main.go` file:
{% highlight go %}
func main() {
    ...

    // here we say we want to create Provider object from `provider` package
    opts := &plugin.ServeOpts{ProviderFunc: provider.Provider}
    ...
    // and execute Proider
    plugin.Serve(opts)

}
{% endhighlight %}

The `provider` package might have another name depending on your provider name (and therefore package name). Usually, the provider package is a separate directory within the whole Provider's repo.

`provider.Provider` is a name of a function within the `provider` package and it MUST return a pointer to `schema.Provider` object in order to conform to Terraform protocol.

In general, a Provider should be defined in the following way:
{% highlight go %}
func Provider() *schema.Provider {
	return &schema.Provider{
		Schema: map[string]*schema.Schema{
			...
			// all provider attributes are defined here
			// for example, values for accessing Cloud provider
			// such as username, password or token
			"username": {
				Type:        schema.TypeString,
				Required:    true,
			},
		},
		ResourcesMap: map[string]*schema.Resource{
			// all resources that are available within the provider are defined here
			"myprovider_resource_name": resourceFunction(),
		},
		DataSourcesMap: map[string]*schema.Resource{
			// all data resources that are available within the provider are defined here
			"myprovider_date_source_name": data_resourceFunction(),
		},
		// function which is called during provider initialization
		// main logic of this function should create client and initialize a client for accessing API
		// e.g. AWS Go SDK;
		// it can also execute different logic based on values provided in schema
		ConfigureContextFunc: providerConfigure,
	}
}

// function which creates and initializes client for accessing Cloud/Service API
func providerConfigure(ctx context.Context, d *schema.ResourceData) (interface{}, diag.Diagnostics) {
	// Warning or errors can be collected in a slice type
	var diags diag.Diagnostics

	// here we read value defined in `username` attribute within provider's schema
	username := d.Get("username").(string)

	// and create client for accessing Cloud or Service provider
	c, err := myclient.NewClient(username)

	// if there is an error during client creation, it's needed to populate `diag` object and return it back
	if err != nil {
		diags = append(diags, diag.Diagnostic{
			Severity: diag.Error,
			Summary:  "Unable to create client",
			Detail:   "Unable to create Cloud client",
		})

		// return nil as client object and error
		return nil, diags
	}

	//return client and err
	return c, diags
}
{% endhighlight %}

That means, we can define our provider in Terraform code in the following way:
{% highlight hcl %}
provider "myprovider" {
  username = "daftkid"
}
{% endhighlight %}

In addition, you SHOULD specify all resources and datasources that are supported by your provider. From the code above, we can make an assumption that using this provider we'll be able to manage the next resources:
- myprovider_resource_name
- myprovider_data_source_name

For example, let's take a look at AWS Provider `internal/provider/provider.go` file where `Provider` object is defined.

{% highlight go %}
// Provider returns a *schema.Provider.
func Provider() *schema.Provider {
	provider := &schema.Provider{
		Schema: map[string]*schema.Schema{
			// here all provider's attributes are defined and you can use them within 
			// provider "aws" {} section in your Terraform code
			"access_key": {
				Type:     schema.TypeString,
				Optional: true,
				Default:  "",
				Description: "The access key for API operations. You can retrieve this\n" +
					"from the 'Security & Credentials' section of the AWS console.",
			},
            ...
			"default_tags": {
				Type:        schema.TypeList,
				Optional:    true,
				MaxItems:    1,
				Description: "Configuration block with settings to default resource tags across all resources.",
				Elem: &schema.Resource{
					Schema: map[string]*schema.Schema{
						"tags": {
							Type:        schema.TypeMap,
							Optional:    true,
							Elem:        &schema.Schema{Type: schema.TypeString},
							Description: "Resource tags to default across all resources",
						},
					},
				},
			},
		    ...
			"profile": {
				Type:     schema.TypeString,
				Optional: true,
				Default:  "",
				Description: "The profile for API operations. If not set, the default profile\n" +
					"created with `aws configure` will be used.",
			},
			"region": {
				Type:     schema.TypeString,
				Optional: true,
				Description: "The region where AWS operations will take place. Examples\n" +
					"are us-east-1, us-west-2, etc.", // lintignore:AWSAT003,
			},
		},

		// defining all data sources which are available within the provider
		DataSourcesMap: map[string]*schema.Resource{
			"aws_acm_certificate": 		acm.DataSourceCertificate(),
			"aws_api_gateway_rest_api": apigateway.DataSourceRestAPI(),
			...

		// defining all resources which can be managed by the provider
		ResourcesMap: map[string]*schema.Resource{
			"aws_accessanalyzer_analyzer": accessanalyzer.ResourceAnalyzer(),
			"aws_account_alternate_contact": account.ResourceAlternateContact(),
			...
		},
	},
}
{% endhighlight %}

So, you need to define the following values:
- define Provider Schema that describes all attributes that can be used by provider itself for initialization and configuration
- list all data sources that will be available within your provider
- list all resources that will be available within your provider

Both data sources and resources MUST be defined as a map of strings mapped to the functions that return resource schemas.

# Resource and Data Source generic structure

At this step, we need to define all data sources and resources which are available in our Provider. In our previous example,
we had the following resources list within the provider definition:
{% highlight go %}
...
ResourcesMap: map[string]*schema.Resource{
	// all resources that are available within the provider are defined here
	"myprovider_resource_name": resourceFunction(),
},
{% endhighlight %}

We need to define `resourceFunction()` somewhere within the package in order to make it work. And make sure that this function returns a pointer to `schema.Resource`. Very similar to the Provider mechanism, right? 😀

Let's take a look at some basic resource function structure:
{% highlight go %}
func resourceFunction() *schema.Resource {
	return &schema.Resource{
		// function that will be used for creating your resource within the Cloud
		CreateContext: resourceCreate,

		// function that will be used for reading your resource state within the Cloud
		ReadContext:   resourceRead,

		// function that will be used for modifying your resource within the Cloud
		UpdateContext: resourceUpdate,

		// function that will be used for deleting your resource from within the Cloud
		DeleteContext: resourceDelete,

		// resource schema, where you defines all attributes that can be specified for your resource in .tf files
		Schema: map[string]*schema.Schema{
			// here you have to define all attributes which can be configured for your resource
			"name" {
				// name is a string value
				Type:     schema.TypeString,
				// name MUST be specified in tf code as a resource cannot be unnamed
				// name is just an attribute example and is not mandatory for each resource
				Required: true
			}
		},

		// object that will be responsible for importing your resources into state file
		Importer: &schema.ResourceImporter{
			State: schema.ImportStatePassthrough,
		},
	}
}
{% endhighlight %}

# CRUD functions

We need to implement CRUD functions specified for `CreateContext`, `ReadContext`, `UpdateContext` and `DeleteContext`.
In most cases, they are following the same logic:
- get values from schema
- convert them into a data structure which is required by Client lib
- pass the data structure into the client call (create, modify or delete)
- process client response and errors if any
- convert values from client response object into schema values

CRUD functions MUST accept the following parameters in order to be compliant with Terraform protocols:
- `ctx context.Context` - Terraform's context, you can ignore this parameter as it's not directly related to resource management;
- `d *schema.ResourceData` - ResourceData object that holds all of the resource attributes defined in `.tf` files;
- `m interface{}` - object, that is returned by `providerConfigure` function from the Provider definition.

Here is an example of `Create` function:
{% highlight go %}
func resourceCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// get client from Provider
	// parameter `m` in most cases are the object that has been returned by Provider Config function
	// we need to cast it into our client's type as `interface{}` is too broad type
	c := m.(*myclient.Client)

	// Warning or errors can be collected in a slice type
	var diags diag.Diagnostics

	// read value for an attribute from schema
	name := d.Get("name").(string)

	// call Create function on client and pass object with required fields
	// in our case it's just a name for simplicity but usually that's
	// a complex structure
	resp, err := c.CreateResource(name)

	// process errors
	if err != nil {
		return diag.FromErr(err)
	}

	// you MUST call `d.SetId` function as it will set unique resource ID for your resource in the state
	// if you did not call this function, each time you're applying Terraform code,
	// it will want to recreate the resource
	d.SetId(strconv.Itoa(o.ID))

	// as a good practice, you SHOULD call Read function in order to read your resource created within the cloud
	// and populate all values for it in our internal Terraform state
	resourceRead(ctx, d, m)

	return diags
}
{% endhighlight %}

Here is an example of `Read` function:
{% highlight go %}
func resourceRead(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// getting the client, the same as for Create function
	c := m.(*myclient.Client)

	// Warning or errors can be collected in a slice type
	var diags diag.Diagnostics

	// get an ID of the resource from the internal state
	id := d.Id()

	// read the resource by Id in the Cloud
	resp, err := c.GetOrder(id)

	// process errors if any
	if err != nil {
		return diag.FromErr(err)
	}

	// get attribute value from response object and set it in the state
	d.Set("name", resp.Name)

	return diags
}
{% endhighlight %}

Here is an `Update` function:
{% highlight go %}
func resourceUpdate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// getting the client, the same as for Create function	
	c := m.(*myclient.Client)

	// get an ID of the resource from the internal state
	id := d.Id()

	// check if attribute value has been changed
	// for example, if you changed "name" attribute value in the Terraform code
	if d.HasChange("name") {
		// if yes, send Update request via the client
		_, err := c.Update(id, name)

		// process errors if any
		if err != nil {
			return diag.FromErr(err)
		}
	}

	// read modified resource back and update values in the state
	return resourceRead(ctx, d, m)
}
{% endhighlight %}

Here is a `Delete` function:
{% highlight go %}
func resourceDelete(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// getting the client, the same as for Create function
	c := m.(*myclient.Client)

	// Warning or errors can be collected in a slice type
	var diags diag.Diagnostics

	// get an ID of the resource from the internal state
	id := d.Id()

	// call client to delete the resource
	err := c.Delete(id)

	// process errors if any
	if err != nil {
		return diag.FromErr(err)
	}

	// d.SetId("") is automatically called assuming delete returns no errors, but
	// it is added here for explicitness.
	d.SetId("")

	return diags
}
{% endhighlight %}

Please note that Terraform will not use our plugin and throw an error until we define at least `Create`, `Read` and `Delete` functions as they are basic ones for the resource management lifecycle.

# How to do initial tests of your provider or resource

Once we defined CRUD functions for our resource, we can start testing our provider and resource itself.

Firstly, you need to `build` your provider into an executable binary. For doing so, please run `go build .` in the root directory of your provider repository.

Go then will try to compile and build everything you've already coded. The process will fail if there are any errors in your code (syntactical, types mismatch, and so on). Please fix all errors and rerun the command.

Then please run `go install`. It will copy your provider's binary into the GOBIN directory.

To start using your Provider locally it's needed to say Terraform where to look for your Provider's binary. As I said in the previous [part of the story][first-part], Terraform will look for providers locally in your workdir first (a directory with your `.tf` files), then it will check `~/.terraform.d` directory and then will try to find a Provider within Terraform Registry. That's obvious, that your Provider binary is not in either place as it's stored in `GOBIN` directory.

> `GOBIN` by default is set to `${GOPATH}/bin`, you can get these values by running `go env | grep 'GOPATH\|GOBIN'`.
{: .notice--success}

Well, we've found where our Terraform Provider binary is located. How to say Terraform look for our Provider within this location?

We can do it via creating a so-called `terraform.rc` file and specify where to locate custom and "in-development" providers.

Please create a `.terraformrc` [file][tfrc] within your `$HOME` directory and populate it with the following lines:

{% highlight json %}
provider_installation {
  dev_overrides {
    "myprovider" = "<path to your GOBIN directory>"
  }
  direct {}
}
{% endhighlight %}

The only thing to do left - is to create `.tf` file with your resource/provider definitions and try to run `terraform plan`.

Test `.tf` file for `myprovider` might look like the following:

{% highlight hcl %}
provider "myprovider" {
  username = "daftkid"
}

resource "myprovider_resource_name" "test" {
  name = "my-test-resource"
}
{% endhighlight %}

At the moment, you are ready to play with it and develop your own basic provider. The Provider's complexity is just limited by your goals and Go Lang skills 🙂.

You can get more details in [Terraform Official docs for Provider development][tf-dev-docs].

[tf]:https://www.terraform.io/
[how-tf-works]:https://www.terraform.io/plugin/how-terraform-works
[tf-reg]:https://registry.terraform.io/
[rpc]:https://en.wikipedia.org/wiki/Remote_procedure_call
[tf-protocol]:https://www.terraform.io/plugin/how-terraform-works#terraform-plugin-protocol
[tfrc]:https://www.terraform.io/cli/config/config-file
[first-part]:/terraform-provider-how-does-it-work/
[tf-dev-docs]:https://www.terraform.io/plugin
