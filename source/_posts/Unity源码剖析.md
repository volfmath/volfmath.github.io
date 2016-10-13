---
title: Unity源码剖析
date: 2016-10-13 04:45:11
tags:
type: "categories"
---

### C++测试框架Catch
#### 什么是Catch
Catch是一个多范式的C++自动测试框架，可用于C++,Objective-c(和C)。他完全用几个头文件来实现，为了易用性，最终打包在一个头文件中。

#### 为什么需要另一个c++测试框架
事实上c++的测试框架已经有很多了，比如[CppUnit](https://sourceforge.net/projects/cppunit/)、[GoogleTest](http://code.google.com/p/googletest/)、 [Boost.Test](http://www.boost.org/doc/libs/1_49_0/libs/test/doc/html/index.html)、[Aeryn](https://launchpad.net/aeryn)、 [Cute](http://r2.ifs.hsr.ch/cute)、 [Fructose](http://fructose.sourceforge.net/) 、[等等](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#C.2B.2B)。
甚至Objective-C的测试框架也有一些，比如现在Xcode中集成的OCUnit.
所以到底Catch和这些相比有什么不同之处呢?除了一个好记的名字之外。
#### 关键功能
- 使用起来非常方便，只需要包含catch.hpp，就可以开始用了
- 不需要额外的依赖，只要你能编译c++98并且有c++标准库
- 写测试用例时，自己注册函数和方法
- 测试用例会被分为各个部分，独立运行(消除了对测试夹具的需求)
- 和传统测试用例一样使用BDD-style Given-When-Then sections
- 只有一个唯一的断言的宏作为比较，并且使用标准C/C++操作符

#### 其他一些功能
- Tests are named using free-form strings - no more couching names in legal identifiers.
- Tests can be tagged for easily running ad-hoc groups of tests.
- Failures can (optionally) break into the debugger on Windows and Mac.
- Output is through modular reporter objects. Basic textual and XML reporters are included. Custom reporters can easily be added.
- JUnit xml output is supported for integration with third-party tools, such as CI servers.
- A default main() function is provided (in a header), but you can supply your own for complete control (e.g. integration into your own test runner GUI).
- A command line parser is provided and can still be used if you choose to provided your own main() function.
Catch can test itself.
- Alternative assertion macro(s) report failures but don't abort the test case
- Floating point tolerance comparisons are built in using an expressive Approx() syntax.
- Internal and friendly macros are isolated so name clashes can be managed
Support for Matchers (early stages)

#### Objective-C的特殊功能
- 自动检测被用于OC项目
- 无论是否使用ARC(自动引用计数)都无需附加的配置
Implement test fixtures using Obj-C classes too (like OCUnit)
Additional built in matchers that work with Obj-C types (e.g. string matchers)

#### 获取Catch
获取Catch最简单的办法是下载最新的头文件。单一的头文件通过独立的一些头文件生成。

Catch的源代码包含测试工程，文档等托管在Github上。 http://catch-lib.net重定向到那里。

在何处部署Catch?
Catch只有头文件。所有需要做的是把文件丢到你项目能找到的地方。可以放在一些你头文件可以搜寻到的核心位置，也可以直接放到你的项目树里！对于其他开源项目想要使用Catch来做测试的话是非常方便的选择。[继续阅读此博客获取更多信息](http://www.levelofindirection.com/journal/2011/5/27/unit-testing-in-c-and-objective-c-just-got-ridiculously-easi.html)

下面教程假定Catch包含单一头文件(或包含文件夹)是不合格的，你需要自己修改下，比如需要的话添加文件夹名。
#### 写测试

我们先从一个简单的范例开始。假设你需要一个函数来计算阶乘，让后你想要测试他(我们暂时先不去考虑TDD)
```c++
unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}
```
为了简单起见我们把所有东西放到一个文件里(后面会介绍如何组织你的测试文件)


```c++

#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"

unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}

TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}

```

这将编译成一个完整的可执行文件，响应命令行。如果不带任何参数的运行，将会执行所有的测试用例（这个例子中只有一个）,报告所有的失败，报告一个有多少测试通过和失败的总结，返回失败测试的数量（用于你只想知道正确与否，是否正常工作的情况）

如果你运行这些代码他将通过测试。一切正常。然而真的如此么？不，这儿任然有个bug。事实上这个教程的第一个版本真的有一个bug在这里，并不是我故意而为之。(感谢Daryle Walker (@CTMacUser) 指出了这点)。

bug是什么？0的阶乘是什么？0的阶乘是1，你应该知道和记住这点。

让我们加到测试用例里:
```c++
TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(0) == 1 );
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

现在我们获得了一个错误，像这样：
```c++
Example.cpp:9: FAILED:
  REQUIRE( Factorial(0) == 1 )
with expansion:
  0 == 1
```

注意到我们得到一个0阶乘的真正的返回值1以及0(函数的结果)，和一个自然的表达式符号 == (等号)。这让我们立刻能知道问题出在哪。
让我们把阶乘函数改成这样：
```c++
unsigned int Factorial( unsigned int number ) {
  return number > 1 ? Factorial(number-1)*number : 1;
}
```
现在所有测试都通过了


当然这里任然有很多问题需要处理。比方说我们碰到一个问题，当返回时突然越界(无符号整形的边界)。阶乘函数这种问题很快会发生。你想为此添加一些测试，来决定怎么处理它们。那让我们停下来处理下这个问题。

我们该怎么做呢？

尽管只是一个简单的测试，但已足以展示Catch如果使用的一些方面。让我们考虑下之前我们做的。

我们定义了(#define)了一个标识，包含了一个头文件，我们就完成了所有的事情，甚至包括一个[带有命令行参数](https://github.com/philsquared/Catch/blob/master/docs/command-line.md)的main()函数。显然，你只能使用那个定义在一个实现文件中。一旦你有多个单元测试文件，你仅仅需要#include "catch.hpp" 。通常一个好的办法是有一个专有的文件来定义(#define)CATCH_CONFIG_MAIN 包含catch.hpp文件。你也可以自己提供main的实现，以自己来驱动Catch，查看[Supplying-your-own-main](https://github.com/philsquared/Catch/blob/master/docs/own-main.md)

我们介绍测试用例使用了TEST_CASE 这个宏。他有1到2个参数，一个自由定义的测试名称和一个可选的，一个或多个标签(更多请查看[Test cases and Sections](https://github.com/philsquared/Catch/blob/master/docs/tutorial.md#test-cases-and-sections))。测试名称必须是唯一的，你可以运行一系列测试通过提供一个通配符测试名称或者一个标签表达式。查看[command line docs](https://github.com/philsquared/Catch/blob/master/docs/command-line.md)获取更多信息。
名称和标签参数都是字符串。我们不需要去强制声明一个函数或方法，或者在任何地方明确的注册测试用例。因为在背后，我们用生成的名字为你定义了一个函数，并且用静态类自动注册了这个函数。通过将函数名称抽象化，我们可以给我们的测试命名而不需要包含标识名。

We write our individual test assertions using the REQUIRE macro. Rather than a separate macro for each type of condition we express the condition naturally using C/C++ syntax. Behind the scenes a simple set of expression templates captures the left-hand-side and right-hand-side of the expression so we can display the values in our test report. As we'll see later there are other assertion macros - but because of this technique the number of them is drastically reduced.

Test cases and sections

Most test frameworks have a class-based fixture mechanism. That is, test cases map to methods on a class and common setup and teardown can be performed in setup() and teardown() methods (or constructor/ destructor in languages, like C++, that support deterministic destruction).

While Catch fully supports this way of working there are a few problems with the approach. In particular the way your code must be split up, and the blunt granularity of it, may cause problems. You can only have one setup/ teardown pair across a set of methods, but sometimes you want slightly different setup in each method, or you may even want several levels of setup (a concept which we will clarify later on in this tutorial). It was problems like these that led James Newkirk, who led the team that built NUnit, to start again from scratch and build xUnit).

Catch takes a different approach (to both NUnit and xUnit) that is a more natural fit for C++ and the C family of languages. This is best explained through an example:
```c++
TEST_CASE( "vectors can be sized and resized", "[vector]" ) {

    std::vector<int> v( 5 );

    REQUIRE( v.size() == 5 );
    REQUIRE( v.capacity() >= 5 );

    SECTION( "resizing bigger changes size and capacity" ) {
        v.resize( 10 );

        REQUIRE( v.size() == 10 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "resizing smaller changes size but not capacity" ) {
        v.resize( 0 );

        REQUIRE( v.size() == 0 );
        REQUIRE( v.capacity() >= 5 );
    }
    SECTION( "reserving bigger changes capacity but not size" ) {
        v.reserve( 10 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "reserving smaller does not change size or capacity" ) {
        v.reserve( 0 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );
    }
}
```
For each SECTION the TEST_CASE is executed from the start - so as we enter each section we know that size is 5 and capacity is at least 5. We enforced those requirements with the REQUIREs at the top level so we can be confident in them. This works because the SECTION macro contains an if statement that calls back into Catch to see if the section should be executed. One leaf section is executed on each run through a TEST_CASE. The other sections are skipped. Next time through the next section is executed, and so on until no new sections are encountered.

So far so good - this is already an improvement on the setup/teardown approach because now we see our setup code inline and use the stack.

The power of sections really shows, however, when we need to execute a sequence of, checked, operations. Continuing the vector example, we might want to verify that attempting to reserve a capacity smaller than the current capacity of the vector changes nothing. We can do that, naturally, like so:
```c++
    SECTION( "reserving bigger changes capacity but not size" ) {
        v.reserve( 10 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 10 );

        SECTION( "reserving smaller again does not change capacity" ) {
            v.reserve( 7 );

            REQUIRE( v.capacity() >= 10 );
        }
    }
```
Sections can be nested to an arbitrary depth (limited only by your stack size). Each leaf section (i.e. a section that contains no nested sections) will be executed exactly once, on a separate path of execution from any other leaf section (so no leaf section can interfere with another). A failure in a parent section will prevent nested sections from running - but then that's the idea.

BDD-Style

If you name your test cases and sections appropriately you can achieve a BDD-style specification structure. This became such a useful way of working that first class support has been added to Catch. Scenarios can be specified using SCENARIO, GIVEN, WHEN and THEN macros, which map on to TEST_CASEs and SECTIONs, respectively. For more details see Test cases and sections.

The vector example can be adjusted to use these macros like so:
```c++
SCENARIO( "vectors can be sized and resized", "[vector]" ) {

    GIVEN( "A vector with some items" ) {
        std::vector<int> v( 5 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );

        WHEN( "the size is increased" ) {
            v.resize( 10 );

            THEN( "the size and capacity change" ) {
                REQUIRE( v.size() == 10 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "the size is reduced" ) {
            v.resize( 0 );

            THEN( "the size changes but not capacity" ) {
                REQUIRE( v.size() == 0 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
        WHEN( "more capacity is reserved" ) {
            v.reserve( 10 );

            THEN( "the capacity changes but not the size" ) {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "less capacity is reserved" ) {
            v.reserve( 0 );

            THEN( "neither size nor capacity are changed" ) {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
    }
}
```
Conveniently, these tests will be reported as follows when run:

Scenario: vectors can be sized and resized
     Given: A vector with some items
      When: more capacity is reserved
      Then: the capacity changes but not the size

Scaling up

To keep the tutorial simple we put all our code in a single file. This is fine to get started - and makes jumping into Catch even quicker and easier. As you write more real-world tests, though, this is not really the best approach.

The requirement is that the following block of code (or equivalent):
```
#define CATCH_CONFIG_MAIN
#include "catch.hpp"
```
appears in exactly one source file. Use as many additional cpp files (or whatever you call your implementation files) as you need for your tests, partitioned however makes most sense for your way of working. Each additional file need only #include "catch.hpp" - do not repeat the #define!

In fact it is usually a good idea to put the block with the #define in it's own source file.

Do not write your tests in header files!

Next steps

This has been a brief introduction to get you up and running with Catch, and to point out some of the key differences between Catch and other frameworks you may already be familiar with. This will get you going quite far already and you are now in a position to dive in and write some tests.

Of course there is more to learn - most of which you should be able to page-fault in as you go. Please see the ever-growing Reference section for what's available.
