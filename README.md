# BlazorCollapsableNavMenu
A simple blazore server-side app to demonstrate collapsable submenus.

## CSS Isolation

.NET Core 5.0 introduced into Blazor a new feature called
[CSS Isolation](https://docs.microsoft.com/en-us/aspnet/core/blazor/components/css-isolation),
which allows Blazor components to have it's own "private" CSS styles.
Subsequently, the Blazor App Visual Studio template contains additional CSS files.
In .NET Core 3.1, all site-specific styles were located in a single site.css file.
In .NET Core 5.0, some styles have been migrated into component-specific css files, such as
MainLayout.razor.css or NavMenu.razor.css.

For the most part, CSS Isolation is a great and helpful feature. But, there are also some
unwanted consequences, which at the surface level can be thought of as removal of c (cascading)
from the CSS acronym. Styles defined in a component-specific css file only apply to
a single component; there is no ineritance mechanism to apply them to child components.
To be exact, there is a new `::deep` combinator that causes a given CSS selector to select
descendant elements as well; however, it can only be applied to individual styles and not the entire
component-specific css file.

Case in point: the BlazorCollapsableNavMenu project refactored the NavMenu component (part of
the Blazor App template) to extract individual menu items (`<li>` elements) into a child
component named NavItem. In .NET Core 3.1, all styles specific to NavMenu component were
automatically applied to NavItem - after all, the styles were defined in a single site.css
file. But, on .NET Core 5.0, CSS Isolation caused all styles defined in NavMenu.razor.css
invisible to NavItem.razor.

Can the scope of styles defined in a component-specific CSS file be applied to other components?
Yes, it can be done by explicitely assigning a (unique) CSS scope to a css file.

> *Note that in absence of such explicit assignment, a CSS scope value is randomly generated by
a compiler. CSS Isolation adds the CSS scope as an extra attribute to html elements; for example,
`<div class="page" b-hvgdpq5shz>`, which is matched by this CSS selector: `.page[b-hvgdpq5shz]`.*

In order for the NavItem component to inherit the styles from the NavMenu component,
both components were assigned a unique CSS scope named "unique-navmenu-cssscope". Specifically,

* Both components need to have their "own" css files, so an empty NavItem.razor.cs file was added next
NavItem.razor file.

* The following section was added to the .csproj file:

``` xml
     <ItemGroup>
       <None Update="Shared/NavMenu.razor.css" CssScope="unique-navmenu-cssscope" />
       <None Update="Components/NavMenu/NavItem.razor.css" CssScope="unique-navmenu-cssscope" />
     </ItemGroup>
```

