[![Build status](https://ci.appveyor.com/api/projects/status/pjruep5140tv84nd)](https://ci.appveyor.com/project/Thwaitesy/mstesthacks)

Overview
==========================================================================
Just a bunch of hacks to get around the deficiencies of MSTest. 

Hopefully for those of us that have to work inside the constrainsts of MSTest, this library should ease our pain. (Just a little) 

Check out the tests project for a few samples

Features
==========================================================================
***Runtime DataSource***

A runtime data driven test as opposed to compile time. Just point your datasource to a property, field or method name that returns an IEnumerable and at runtime it will loop through the collection and just act like normal. (Think NUnit's [TestCaseSource](http://nunit.org/index.php?p=testCaseSource&r=2.5))

***Exception Assert***

Provides a much easier way to throw exceptions and verify the type, and information within the exception itself.

Getting Started
==========================================================================
[Install NuGet](http://docs.nuget.org/docs/start-here/installing-nuget). Then, install MSTestHacks from the package manager console:
```csharp
PM> Install-Package MSTestHacks
``` 

###Runtime DataSource
*Note:* App.config or Web.config must be in the project to provide the dynamic linking.

**1)** You MUST inherit your test class from `TestBase`
```csharp
[TestClass]
public class UnitTest1 : TestBase
{ }
```

**2)** Create a Property, Field or Method, that returns an IEnumerable<T>
```csharp
[TestClass]
public class UnitTest1 : TestBase
{
    private IEnumerable<int> Stuff
    {
        get
        {
            //This could do anything, fetch a dynamic list from anywhere....
            return new List<int> { 1, 2, 3 };
        }
    }
}
```

**3)** Add the `DataSource` attribute to your test method, pointing back to the IEnumerable<T> name created earlier. This needs to be fully qualified.
```csharp
[TestMethod]
[DataSource("Namespace.UnitTest1.Stuff")]
public void TestMethod1()
{
    var number = this.TestContext.GetRuntimeDataSourceObject<int>();
    
    Assert.IsNotNull(number);
}
```
**Explanation**
When each TestBase inherited class is initialised, a process gets run to create an XML file for each datasource attribute. It then dynamically links each datasource up to the XML file. So 
when each test executes it loops over the datasource like it would normally in MSTest. The "GetRuntimeDataSourceObject" extension method is just a convenient helper to get 
the object back out of the datasource using JSON deserialisation. Simple really :)

###Exception Assert
```csharp
//Verify the type and message.
[TestMethod]
public void MethodThrowsSpecificExceptionWithExpectedExceptionMessage()
{
	var expectedMessage = "crap.";
	var action = new Action(() => { throw new StackOverflowException(expectedMessage); });

	ExceptionAssert.Throws<StackOverflowException>(action, expectedMessage);
}

//Verify the type and anything about the exception with the 'validator'
[TestMethod]
public void MethodThrowsExceptionAndTheValidatorWorks()
{
	// Arrange
	Action action = () => { throw new Exception("This is silly"); };

	// Act & Assert
	ExceptionAssert.Throws<Exception>(action, validatorForException: x => x.Message == "This is silly");
}
```

Changelog
==========================================================================
*2.3.1*
- Added support for .net 40 framework

*2.2.19*
- Added Exception Assert Feature
- Removed CodedUI Support (See [AFrame](https://github.com/Thwaitesy/AFrame))
 
*2.1.0*
- Made references to MS TestTools not point to "Specific" versions e.g. VS2012 references

*2.0.0*
- More logging for runtime datasource around timing etc
- Introduced CodedUI Jquery controls for finding controls via jquery selectors

*1.1.2*
- Added a fix so the datasources could point to an class that didnt inherit TestBase

*1.1.1*
- Removed the need for [AttachRuntimeDataSources(typeof(ClassName))] - dynamically finds the datasources.
- Added some rudimentary debugging
- Moved all outputed data down 1 directory to be self contained in a directory called MSTestHacks
- GetRuntimeDataSourceObject<T> extenstion method is now in same namespace as testbase, so no need now for the using statement: `using MSTestHacks.RuntimeDataSources;` 
- Made code a little more efficient

*1.1.0 - (BREAKING CHANGE)*
- Creating a datasource file per datasource, simplifies life
- All datasources now have to be fully qualified, pointing to the IEnumerable<T>. This creates a unique datasource that was required in some instances. 

*1.0.2*
- Inject the ConnectionString automatically
- Add a couple more tests

*1.0.1*
- Fixes the issue with stale iterations. Each run was not getting deleted from the xml file and was building up. 

*1.0.0*
- Replaced the database backend with an xml one
- Production Ready

*0.0.2*
- Simplified NuGet and added project links etc

*0.0.1*
- Initial release

Contributors
==========================================================================
Sam Thwaites    
Corey Warner

Licence
==========================================================================
See [LICENCE](https://github.com/Thwaitesy/MSTestHacks/blob/master/LICENCE)
