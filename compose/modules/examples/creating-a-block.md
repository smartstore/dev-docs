# 🥚 Creating a Block

A `Block` is used to create pages quickly within the Page Builder. It can take on various forms:

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

Now you can use your `Block` in the Page Builder.

## Advanced topics

### Using a widget

## Conclusion

We learned a lot here today...

{% hint style="info" %}
For more examples, take a look at the folder `Blocks` in the Page Builder module.
{% endhint %}

Download file:

\<insert file here>
