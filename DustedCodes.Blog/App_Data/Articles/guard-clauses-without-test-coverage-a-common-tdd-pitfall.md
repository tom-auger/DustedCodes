﻿<!--
    Published: 2015-03-19 23:28
    Author: Dustin Moris Gorski
    Title: Guard clauses without test coverage, a common TDD pitfall
    Tags: tdd guard-clauses
-->
<p>Today I wanted to blog about a little mistake which easily creeps into your otherwise good TDD practices.</p>

<p>For this demo I'd like to start with an empty unit test project. I start initializing a subject under test and then implementing just enough code to make the solution build:</p>
<pre><code>namespace MyClassLibrary
{
    public class DomainClass { }

    [TestFixture]
    public class DomainClassTests
    {
        [Test]
        public void Test1()
        {
            var sut = new DomainClass();
        }
    }
}</code></pre>

<p>For this example the method under test will have some very trivial business logic:</p>
<ol>
    <li>Accept an argument</li>
    <li>Check if the argument is NULL and return early, otherwise proceed</li>
    <li>Call into a dependency</li>
    <li>Return a result</li>
</ol>

<p>Let's finish the first test by checking for the simple case of a NULL argument:</p>
<pre><code>[TestFixture]
public class DomainClassTests
{
    [Test]
    public void DoSomeWork_With_Null_Input_Will_Return_Null()
    {
        // Arrange
        var sut = new DomainClass();

        // Act
        var actual = sut.DoSomeWork(null);

        // Assert
        Assert.IsNull(actual);
    }
}</code></pre>

<p>In true TDD practise you'd write just enough code to compile the project first, then let the test fail and at last return null to make it pass:</p>
<pre><code>public class DomainClass
{
    public object DoSomeWork(object arg)
    {
        return null;
    }
}</code></pre>

<p><strong>However, at this point I have often seen developers implememnt a little bit more code than actually required by the test:</strong></p>
<pre><code>public class DomainClass
{
    public object DoSomeWork(object arg)
    {
        // This guard clause has not been enforced by a particular unit test yet:
        if (arg == null)
            return null;
        
        return input;
    }
}</code></pre>

<p>It is very tempting to write a bit more code if you already know what the desired end result should look like. Guard clauses are so trivial that it makes it a very common mistake.</p>
<p>Someone might argue that the code is justified, because if you comment out the if statement the test will fail.</p>
<p>This is not true though, because the entire condition as a whole is not required yet and as we have proven above, a simple <code>return null;</code> statement satisfies the test as well.</p>

<p>This little mistake seems very harmless now, but it could inroduce a potential NullReferenceException later in the project when the first developer starts to re-factor code and remove redundant code.</p>

<p>This will essentially happen when we continue implementing the remaining unit tests and business logic for the method under test:</p>

<pre><code>public object DoSomeWork(object arg)
{
    if (arg == null)
        return null;
    
    var result = _dependecy.ProcessData(arg);
    return result;
}</code></pre>

<p>Now at this point I could delete the if statement together with the &quot;return null&quot; and the first test will still succeed:</p>
<pre><code>public object DoSomeWork(object arg)
{
    // if (input == null)
    //    return null;
    
    var result = _dependecy.ProcessData(arg);
    return result;
}</code></pre>

<p>Why? Well because in the first test we didn't set up a stub for the dependency yet:</p>
<pre><code>[TestFixture]
public class DomainClassTests
{
    [Test]
    public void DoSomeWork_With_Null_Input_Will_Return_Null()
    {
        // Arrange
        var sut = new DomainClass();

        // Act
        var actual = sut.DoSomeWork(null);

        // Assert
        Assert.IsNull(actual);
    }
}</code></pre>

<p>It means that &quot;<code>var result = _dependecy.ProcessData(arg);</code>&quot; will return null and make the test go green.</p>

<p>This is why it is best to not introduce little short cuts in TDD and be careful during your code reviews as well!</p>