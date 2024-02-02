---
outline: deep
---

# View tag helpers

View tag helpers are an extension built in into Hydro that let you create new kind of view components (called view tag helpers) that can replace partials, editors and regular view components.

Here is a basic example of a view tag helper called `Submit`:

```c#
// Submit.cshtml.cs

public class SubmitTagHelper : HydroTagHelper;
```

```razor
<!-- Submit.cshtml -->

@model SubmitTagHelper

<button class="btn btn-primary" type="submit">
  Save
</button>
```

Now we can use our tag helper in any other razor view:

```razor
<!-- SomeForm.cshtml -->

<submit />
```

## Naming conventions

View tag helpers follow the ASP.NET Mvc Tag Helpers naming conventions, which means when
the view tag helper has suffix `TagHelper`, the prefix in kebab-case will be used as the tag name. Example:

- `AlertTagHelper` gives `<alert />`.
- `SubmitButtonTagHelper` gives `<submit-button />`.

It's possible to not use the default convention and specify the tag names by using `[HtmlTargetElement]` attribute, for example:

```c#
// SubmitButton.cshtml.cs

[HtmlTargetElement(nameof(SubmitButton))]
public class SubmitButton : HydroTagHelper;
```

Usage:

```razor
<SubmitButton />
```

## Parameters

View tag helpers use parameters to pass the data from a caller to the view. Example:

```c#
// Alert.cshtml.cs

public class AlertTagHelper : HydroTagHelper
{
    public string Message { get; set; }    
}
```

```razor
<!-- Alert.cshtml -->

@model AlertTagHelper

<div class="alert">
  @Model.Message
</div>
```

Now we can set a value on the `message` attribute that will be passed to our `AlertTagHelper`:

```razor
<!-- Index.cshtml -->

<alert message="Success" />
```

Parameter names are converted to kebab-case when used as attributes on tags, so:
- `Message` property becomes `message` attribute.
- `StatusCode` property becomes `status-code` attribute.

## Dynamic attributes

All attributes passed to the view tag helper by the caller are available in a view definition, even when they are not defined as properties.
We can use this feature to pass optional html attributes to the view, for example:

```c#
// Alert.cshtml.cs

public class AlertTagHelper : HydroTagHelper
{
    public string Message { get; set; }    
}
```

```razor
<!-- Alert.cshtml -->

@model AlertTagHelper

<div class="alert @Model.Attribute("class")">
  @Model.Message
</div>
```

Now we can set an optional attribute `class` that will be added to the final view:

```razor
<!-- Index.cshtml -->

<alert message="Success" class="alert-green" />
```

## Child content

View tag helpers support passing the child html content, so it can be used later when rendering the tag. Example:

```c#
// DisplayField.cshtml.cs

public class DisplayFieldTagHelper : HydroTagHelper
{
    public string Title { get; set; };
}
```

```razor
<!-- DisplayField.cshtml -->

@model DisplayFieldTagHelper

<div class="display-field">
  <div class="display-field-title">
    @Model.Title
  </div>

  <div class="display-field-content">
    @Model.Slot()
  </div>
</div>
```

Usage:

```razor
<!-- SomePage.cshtml -->

<display-field title="Price">
  <i>199</i> EUR
</display-field>
```

Remarks:
- `Model.Slot()` renders the child content passed by the tag caller


## Slots

Slots are placeholders for html content inside view tag helpers that can be passed by the caller. Here is an example of a `Card` tag:

```c#
// Card.cshtml.cs

public class CardTagHelper : HydroTagHelper;
```

```razor
<!-- Card.cshtml -->

@model CardTagHelper

<div class="card">
  <div class="card-header">
    @Model.Slot("header")
  </div>

  <div class="card-content">
    @Model.Slot()
  </div>

  <div class="card-footer">
    @Model.Slot("footer")
  </div>
</div>
```

Usage:

```razor
<!-- SomePage.cshtml -->

<card>
  <slot name=“header”>
    <strong>Laptop</strong>
  </slot>

  Information about the product

  <slot name=“footer”>
    <i>Price: $199</i>
  </slot>
</card>
```

Remarks:
- `Model.Slot("header")` renders the content of passed through `<slot name=“header”>`
- `Model.Slot("footer")` renders the content of passed through `<slot name=“footer”>`
- `Model.Slot()` renders the rest of the child content

## Differences between Hydro components and view tag helpers

**View tag helpers:**
- Used to render views.
- Not stateful or interactive.
- Replacement for partial views, editors or regular view components.
- Rerendered on each request.

**Hydro components:**
- Used to render functional components.
- Stateful and interactive.
- Should be used when state is needed.
- Rerendered only in specific situations, like action calls or events.