# 🥚 Validation

## FluentValidation

Smartstore uses [FluentValidation](https://fluentvalidation.net/) for building strongly-typed validation rules to validate view models on the server-side. For this purpose, rules are defined for properties of the related view model. If the value of a property is changed via an edit page, it will be validated against its rule and, if necessary an error message is issued if the rule is not fulfilled. Typically, the validation class is located directly below the view model in the same file as the model.

The class of a validator inherits from `AbstractValidator`, to which the view model type is passed as a generic parameter.

```csharp
public partial class CustomerValidator : AbstractValidator<CustomerModel>
{
    public CustomerValidator(CustomerSettings customerSettings)
    {
        RuleFor(x => x.Password).NotEmpty().When(x => x.Id == 0);

        if (customerSettings.FirstNameRequired)
            RuleFor(x => x.FirstName).NotEmpty();

        if (customerSettings.LastNameRequired)
            RuleFor(x => x.LastName).NotEmpty();

        if (customerSettings.CompanyRequired && customerSettings.CompanyEnabled)
            RuleFor(x => x.Company).NotEmpty();

        if (customerSettings.PhoneRequired && customerSettings.PhoneEnabled)
            RuleFor(x => x.Phone).NotEmpty();

        // Further code has been omitted for clarity.
    }
}
```

FluentMigrator provides a number of ways to specify complex validation rules.

```csharp
public AddressValidator(Localizer T, AddressSettings addressSettings)
{
    // Validate email address.
    RuleFor(x => x.Email).EmailAddress();

    if (addressSettings.ValidateEmailAddress)
    {
        // Validate additional input field to confirm the entered e-mail address.
        RuleFor(x => x.EmailMatch)
            .NotEmpty()
            .EmailAddress()
            .Equal(x => x.Email)
            .WithMessage(T("Admin.Address.Fields.EmailMatch.MustMatchEmail"));
    }
}
```

```csharp
public ConfigurationValidator(Localizer T)
{
    // Validate a list of entered IP addresses.
    RuleFor(x => x.WebhookIps)
        .Must(ips =>
        {
            if (ips != null)
            {
                foreach (var ip in ips)
                {
                    if (!IPAddress.TryParse(ip, out _))
                    {
                        return false;
                    }
                }
            }

            return true;
        })
        .WithMessage(T("Plugins.SmartStore.PostFinance.InvalidIp"));
}
```

### SmartValidator

Typically view models properties have the same names as the corresponding object that is to be modified (e.g. an entity such as a category or a settings class). This way the values of the properties can be copied from an object to its model and vice versa by a single `MiniMapper` statement. This allows Smartstore to provide additional utilities.

`SmartValidator.ApplyEntityRules` copies common validation rules from the entity type over to the corresponding view model type. Common rules are `Required` and `MaxLength` rules on string properties (either fluently mapped or annotated). It also adds the `Required` rule to non-nullable intrinsic model property type to bypass MVC's non-localized `RequiredAttributeAdapter`.

```csharp
public partial class CategoryValidator : SmartValidator<CategoryModel>
{
    public CategoryValidator(SmartDbContext db)
    {
        ApplyEntityRules<Category>(db);
    }
}
```

TIP: Validation errors and support requests can be avoided by automatically trimming certain data before saving it. Especially when it comes to data that never starts and ends with spaces, such as data for an API access, a bank account or passwords:

```csharp
model.ApiPassword = model.ApiPassword.TrimSafe();
model.ApiSecretWord = model.ApiSecretWord.TrimSafe();
MiniMapper.Map(model, settings);
```

### SettingModelValidator

`SettingModelValidator` is an abstract validator that is capable of ignoring rules for unchecked setting properties in a store-specific edit session.

```csharp
public class SearchSettingValidator : SettingModelValidator<SearchSettingsModel, SearchSettings>
{
    public SearchSettingValidator(Localizer T)
    {
        RuleFor(x => x.InstantSearchNumberOfProducts)
            .Must(x => x >= 1 && x <= 16)
            .WhenSettingOverriden((m, x) => m.InstantSearchEnabled)
            .WithMessage(T("Admin.Validation.ValueRange", 1, 16));
    }
}
```

The rule validates `InstantSearchNumberOfProducts` with a value between 1 and 16 under the following conditions:

* In case of store-agnostic setting mode: the result of the predicate parameter returns true, OR
* In case of store-specific setting mode: validated setting `InstantSearchNumberOfProducts` is overriden for current store.

HINT: `SettingModelValidator` does not support manually validation using `IValidator` directly.

### Validate manually

Use `IValidator` to manually validate a view model.

```csharp
[SystemName("Payments.IPaymentCreditCard")]
[FriendlyName("ipayment Credit Card")]
public class CreditCardProvider : IonosProviderBase<IonosPaymentCreditCardSettings>, IConfigurable
{
    private readonly IValidator<CCPaymentInfoModel> _validator;
    
    public override async Task<PaymentValidationResult> ValidatePaymentDataAsync(IFormCollection form)
    {
        var model = new CCPaymentInfoModel
        {
            CardholderName = form["CardholderName"],
            CardNumber = form["CardNumber"],
            CardCode = form["CardCode"]
        };

        var result = await _validator.ValidateAsync(model);
        return new PaymentValidationResult(result);
    }
}
```

## MVC model validation <a href="#model-validation-in-aspnet-core-mvc-and-razor-pages" id="model-validation-in-aspnet-core-mvc-and-razor-pages"></a>

Since Smartstore is based on ASP.NET Core MVC, the server-side and client-side validations of this framework are also available.

### Server-side

The model state represents errors that come from the MVC model binding or from model validation. If the model state is not valid (i.e. it contains errors), then no data should be saved and instead the edit page should be called again including the model state errors.

```csharp
[AuthorizeAdmin, Permission(DevToolsPermissions.Read), LoadSetting]
public IActionResult Configure(ProfilerSettings settings)
{
    var model = MiniMapper.Map<ProfilerSettings, ConfigurationModel>(settings);
    return View(model);
}

[HttpPost, AuthorizeAdmin, Permission(DevToolsPermissions.Update), SaveSetting]
public IActionResult Configure(ConfigurationModel model, ProfilerSettings settings)
{
    if (!ModelState.IsValid)
    {
        return Configure(settings);
    }

    ModelState.Clear();
    MiniMapper.Map(model, settings);

    return RedirectToAction(nameof(Configure));
}
```

HINT: In the case of an invalid model state, the GET Configure method must be called directly, without redirecting, otherwise the validation errors would be lost.

### Client-side

TODO....
