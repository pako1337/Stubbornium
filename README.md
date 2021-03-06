# Stubbornium  ![NuGet](https://img.shields.io/nuget/v/Stubbornium.svg?maxAge=2592000)

Stubbornium is a .NET wrapper for Selenium API that enables you to write robust and stable test for complex sites.

Do your test runs randomly fail because some element was missing? Do you have `Thread.Sleep` call in your tests? Do you insert code for waiting for ajax completion in hope it will make test more stable?

Stubbornium will give you a clean and simple interface for writing tests without worrying about all those aspects. 

## Install

You can install the package from NuGet:

```posh
Install-Package Stubbornium
```

## Principles

Stubbornium implmentes following principles and hides them behind it's API:

- Each interaction with page should define expected result (i.e. *"After I click the button a popup appears"*).
- After the interaction, test should automatically wait for expected result.
- If interaction fails, it should be repeated.
- If expected condition is not met, interaction should be repeated.
- Test should fail if repeating several times didn't help.

# Basic usage

After you initialize `IWebDriver` instance, you can wrap it with Stubbornium:

```csharp
var browser = new StubbornBrowser(WebDriver);
```

To find an element, use `Find` property:

```csharp
browser.Find.TagName("h1")
```

*There's a notable difference between finding element with stubboriun and selenium. Selenium evaluates element immiediatly, where Stubboriun only saves the search pattern and will evaluate it when performing actions on the element (Click, Assert etc)*

## Assertions

Assert method is used to check if element is in desired state. If it's not, or if even it doesn't exist Stubboriun will reevaluate the assertion for several seconds before throwing exception.

Example: verifying element's content:

```csharp
browser
    .Find.TagName("h1")
    .Assert(element => element().Text == "Test", "H1 contains \"Test\"");
```
Notice that parameter `element` in lambda is not the actual selenium `IWebElement` but a `Func<IWebElement>` that let you evaluate the element by calling it. Stubbornium will not evaluate the Element until this function is called.

There's a shorthand method for asserting text:

```csharp
browser
    .Find.TagName("h1")
    .AssertHasText("Test");
```

## Clicking

When performing any action on the element, Stubbornium requires to define what should happen after the action. This information is used to call Selenium wait but also to reiterate if the condition is not met. Similiary to assertions, Stubbornium tries to perform the failing action several times before throwing an exception.

Example: clicking an element (i.e. to change it's content):

```csharp
browser
    .Find.Id("change-title")
    .Click(_ => browser.FindElement(By.TagName("h1")).Text == "Hello");
```

Again there's a shorthand version that does exactly the same thing:

```csharp
browser
    .Find.Id("change-title")
    .ClickToAppear(By.TagName("h1"), e => e.Text == "Hello");
```

`ClickToAppear` returns the WebElement that is expected to appear. This makes it possible to chain the calls. One of the most commonly used scenario is closing popups. In this example first the `change-title` is clicked and then the `reset-title` is clicked:

```csharp    
browser
    .Find.Id("change-title")
    .ClickToAppear(By.Id("reset-title"))
    .ClickToClose();
```
        
## Setting input values

`SetText` can be used to manipulate inputs. Like in all cases, the Stubbornium verifies that the text was indeed set as expected:

```csharp
browser
    .Find.Id("new-title")
    .SetText("Better title");
```

## Hooking to actions

Global static `StubbornConfiguration.Default` can be used to set various settings used thoughout the library. For example, Stubbornium gives an option to run code before every action (`Click`, `Assert` etc). 

To wait for loading element to dissapear before executing action:

```csharp
StubbornConfiguration.Default.BeforeDoActions.Add(driver =>
{
    var wait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));
    wait.Until(ExpectedConditions.InvisibilityOfElementLocated(By.Id("loading")));
});
```

There are some predefined actions. The shorthand for above code is:

```csharp
StubbornConfiguration.Default.WaitForLoaderFinish(By.Id("loading"));
```

To wait for jQuery AJAX calls to complete you can use:

```csharp
StubbornConfiguration.Default.WaitForAjax()
```
## Working samples

You can find above and additional examples in the [Stubbornium.Sample](https://github.com/mandrek44/Stubbornium/tree/master/Stubbornium.Sample) project in Stubbornium repository.

# Log output

Stubbornium creates a detailed log of performed actions. To enable logging, you must configure logger:

```csharp
StubbornConfiguration.Default.Log = new ConsoleLogger();
```

Now when running tests Stubbornium will output information about each step, i.e:

```
Assert - By.TagName: h1 - Has text "Test"
SetText - By.Id: new-title - "Better title"
Click - By.Id: change-title
Assert - By.TagName: h1 - Has text "Better title"
```


# Building from sources

To build Stubbornium use `build.cmd`.

To show all possible targets:
```
build.cmd
```

To build project:
```
build.cmd Build
```

Optionally, you can support `Configuration` parameter:
```
build.cmd Build Configuration=Debug
```
