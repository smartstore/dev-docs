# 🥚 Creating a Block

A `Block` is used to create content quickly within the Page Builder. Various forms have already been implemented by Smartstore like HTML output, images, iframes, product listings and many more. The possibilities for new Blocks are endless.

{% hint style="info" %}
If you want to take a deep dive into the topic, see [Blocks](../../../framework/content/page-builder-and-blocks.md).
{% endhint %}

{% hint style="info" %}
You can find the source code of a sample Block at _Smartstore.DevTools/Blocks/SampleBlock.cs_
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

This is where you add block-settings and values you might need for model preparation. Treat it like a model.

```csharp
[LocalizedDisplay("Plugins.MyOrg.HelloWorld.Name")]
public string Name { get; set; }
```

If you don't want a property to be saved to the database, use `IgnoreDataMember`.

```csharp
[IgnoreDataMember]
public Object NotForTheDatabase { get; set; }
```

Use `BindNever` when manually binding the model.

```csharp
[IgnoreDataMember, BindNever]
public MediaFileInfo mediaFile { get; set; }
```

### Add a BlockHandler

Next add the new class `HelloWorldBlockHandler` to your file and add the interface `BlockHandlerBase<T>` to it.

```csharp
public class HelloWorldBlockHandler : BlockHandlerBase<HelloWorldBlock>
{
    // Doing nothing means standard behaviour.
}
```

Place the following attribute above your `BlockHandler` definition.

```csharp
[Block("helloworld", Icon = "fa fa-eye", FriendlyName = "Hello World")]
```

This does three things:

* `helloworld`: Defines the [location of the views](creating-a-block.md#add-some-views) for your `Block`. A lowercase value is recommended by our guidelines.
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

Here you can define your rules using `FluentValidation`. Add the following to make sure `Name` is never empty.

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
        // Doing nothing means standard behaviour.
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

### Add Views

Create the folders _BlockTemplates/helloworld/_ in the _Views/Shared/_ folder of your module.

{% hint style="warning" %}
The folder name _helloworld_ corresponds to the attribute you set in your [BlockHandler](creating-a-block.md#add-a-blockhandler)!
{% endhint %}

Create the two files _Edit.cshtml_ and _Public.cshtml_ for simple output.

{% hint style="info" %}
Add the reference `@using MyOrg.HelloWorld.Blocks` to \_ViewImports. That way you can access the `HelloWorldBlock` model in all your views.
{% endhint %}

#### Edit.cshtml

This is your configuration view. It functions the same way your _Configure.cshtml_ file works.

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

#### Public.cshtml

This is what your customers will see. You can also see this clicking on **Preview** in the Page Builder.

```cshtml
@model HelloWorldBlock

<div>My block name: @Model.Name</div>
```

{% hint style="info" %}
Create _Preview.cshtml_ to access the grid placement view. It will be shown whilst moving around blocks in the Page Builder.
{% endhint %}

Now you can use your `Block` in the Page Builder.

## Advanced topics

Taking a closer look at the `BlockHandler`, there is some more functionality you can add to your module.

### Handling StoryViewMode

Using `StoryViewMode`, you are able to distinguish between the four different view modes of a `Block`:

* `Edit`: Shown when you edit `Block` properties.
* `GridEdit`: Shown when arranging Blocks on the grid.
* `Preview`: Shown when previewing your story.
* `Public`: Shown in frontend, when your story is published.

Create a new block-setting `MyLocalVar` in `HelloWorldBlock`.

```csharp
public string MyLocalVar { get; set; } = "Initialised in Block";
```

Add the `LoadAsync` function to `HelloWorldBlockHandler`.

```csharp
public override async Task<HelloWorldBlock> LoadAsync(IBlockEntity entity, StoryViewMode viewMode)
```

To access the current block, use the following line.

```csharp
var block = base.Load(entity, viewMode);
```

Now you can use `viewMode` to get the currently used view mode for your block.

```csharp
if (viewMode == StoryViewMode.Edit)
{
    // This only gets called in Edit-Mode
    block.MyLocalVar += " - Running in Edit-Mode";
}else if (viewMode == StoryViewMode.Preview)
{
    // This only gets called in Preview-Mode
    block.MyLocalVar += " - Running in Preview-Mode";
}else if (viewMode == StoryViewMode.GridEdit)
{
    // This only gets called in Grid-Edit-Mode
    block.MyLocalVar += " - Running in Grid-Edit-Mode";
}else if (viewMode == StoryViewMode.Public)
{
    // This only gets called in Public-Mode
    block.MyLocalVar += " - Running in Public-Mode";
}

return block;
```

Insert the following in Edit.cshtml, Preview.cshtml and Public.cshtml.

```cshtml
<div>My local variable: @Model.MyLocalVar</div>
```

{% hint style="info" %}
Use Preview.cshtml, to be able to access Grid-Edit-Mode.
{% endhint %}

Run your code and see how `MyLocalVar` reacts to each view mode.

### Using a widget as a view

## Conclusion

We learned a lot here today...

The code for this module can be downloaded here:

{% file src="../../../.gitbook/assets/MyOrg.HelloWorldBlock.zip" %}
