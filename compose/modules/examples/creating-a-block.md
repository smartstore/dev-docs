# 🥚 Creating a Block

A `Block` is used to create content quickly within the Page Builder. It can take on various forms:

* HTML Output
* An Image
* An iFrame
*   Product lists

    ...

{% hint style="info" %}
If you want to take a deep dive into the topic, see [Blocks](../../../framework/content/page-builder-and-blocks.md).
{% endhint %}

## Write a simple Block

First you create a folder _Blocks_ and add a new file called `HelloWorldBlock.cs`.

### Implement IBlock

Then you add the interface `IBlock`.

```csharp
namespace MyOrg.HelloWorld.Blocks
{
    public class HelloWorldBlock : IBlock
    {
    }
}
```

This is where you add settings and other values. Treat it like a model.

```csharp
[LocalizedDisplay("Plugins.MyOrg.HelloWorld.Name")]
    public string Name { get; set; }
```

If you don't want a property to be saved to the database, use `IgnoreDataMember`.

```csharp
[IgnoreDataMember]
    public Object NotForTheDatabase { get; set; }
```

{% hint style="danger" %}
For all wanting to use _model binding_. Do not forget to add the attribute `IgnoreDataMember` in possible combination with `BindNever`! If you don't, the whole model will be saved to the database.

See _Modules/Smartstore.Blog/Blocks/BlogBlock.cs_ and _Modules/Smartstore.PageBuilder/Blocks/VideoBlock.cs_ for examples.
{% endhint %}

### Add a BlockHandler

Next add the new class `HelloWorldBlockHandler` to your file and add the interface `BlockHandlerBase<T>` to it.

```csharp
public class HelloWorldBlockHandler : BlockHandlerBase<HelloWorldBlock>
{
    //Doing nothing means standard behaviour.
}
```

Place the following attributes above your `BlockHandler` definition.

```csharp
[Block("helloworld", Icon = "fa fa-eye", FriendlyName = "Hello World")]
```

This does three things:

* `helloworld`: Defines the [location of the views](creating-a-block.md#add-some-views) for your `Block`.
* `fa fa-eye`: Defines the _Font Awesome_ icon shown next to your label in the Page Builder.
* `Hello World`: Defines the label text shown in the Page Builder.

You don't need to add any code to your `BlockHandler`. It will have standard behaviour, which is desired at this point. See [Advanced topics](creating-a-block.md#advanced-topics) for other use.

### Add a validator

To make sure all user inputs are correct, add a validator to your file and add the `AbstractValidator<T>` interface.

```csharp
public partial class HelloWorldBlockValidator : AbstractValidator<HelloWorldBlock>
{
    public HelloWorldBlockValidator()
    {
    }
}
```

Here you can define your rules. Add the following to make sure `Name` is never empty.

```csharp
RuleFor(x => x.Name).NotEmpty();
```

If you have more properties you want to check, simply add more rules.

```csharp
RuleFor(x => x.Name).NotEmpty();
RuleFor(x => x.ASmallValue).GreaterThan(0);
RuleFor(x => x.AShortText).MaximumLength(255);
```

Your final code should look something like this:

{% code title="HelloWorldBlock.cs" %}
```csharp
using FluentValidation;
using Smartstore.Core.Content.Blocks;
using Smartstore.Web.Modelling;

namespace MyOrg.HelloWorld.Blocks
{
    [Block("helloworld", Icon = "fa fa-eye", FriendlyName = "Hello World")]
    public class HelloWorldBlockHandler : BlockHandlerBase<HelloWorldBlock>
    {
        //Doing nothing means standard behaviour.
    }
    public class HelloWorldBlock : IBlock
    {
        [LocalizedDisplay("Plugins.MyOrg.HelloWorld.Name")]
        public string Name { get; set; }
    }
    public partial class HelloWorldBlockValidator : AbstractValidator<HelloWorldBlock>
    {
        public HelloWorldBlockValidator()
        {
            RuleFor(x => x.Name).NotEmpty();
        }
    }
}
```
{% endcode %}

### Add some Views

Create the folders _BlockTemplates/helloworld/_ in the _Views/Shared/_ folder of your module.

{% hint style="warning" %}
The folder name _helloworld_ corresponds with the attribute you set in your [BlockHandler](creating-a-block.md#add-a-blockhandler)!
{% endhint %}

Create three files, Edit.cshtml, Preview.cshtml and Public.cshtml for simple output.

{% hint style="info" %}
Add the reference `@using MyOrg.HelloWorld.Blocks` to \_ViewImports. That way you can access the `HelloWorldBlock` model in all your views.
{% endhint %}

#### Edit.cshtml

This is your configuration view. It functions the same way your Configure.cshtml file works.

```cshtml
@model HelloWorldBlock

<div class="adminContent">
    <div class="adminRow">
        <div class="adminTitle">
            <smart-label asp-for="Name" />
        </div>
        <div class="adminData">
            <setting-editor asp-for="Name"></setting-editor>
            <span asp-validation-for="Name"></span>
        </div>
    </div>
</div>
```

#### Preview.cshtml

This is the grid placement view. It is shown whilst moving around the blocks in the Page Builder.

```cshtml
@model HelloWorldBlock

This is a HelloWorld-Block preview.
```

#### Public.cshtml

This is what your customers will see. You can also see this clicking on **Preview** in the Page Builder.

```cshtml
@model HelloWorldBlock

<div>My block name: @Model.Name</div>
```

Now you can use your `Block` in the Page Builder.

## Advanced topics

### Handling StoryViewMode

### Using a widget as a view

## Conclusion

We learned a lot here today...

{% hint style="info" %}
For more examples, take a look at the folder `Blocks` in the Page Builder module.
{% endhint %}

The code for this module can be downloaded here:

{% file src="../../../.gitbook/assets/MyOrg.HelloWorldBlock.zip" %}
