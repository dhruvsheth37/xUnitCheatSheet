# xUnitCheatsheet

xUnit.net is a free and open-source unit testing tool for the .NET Framework, written by the original author of NUnit. It is licensed under Apache License 2.0 and the source code is available on GitHub. xUnit.net works with Xamarin, ReSharper, CodeRush, and TestDriven.NET.


```C#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;
using Xunit;
using Xunit.Sdk;

namespace xUnitCheatsheet.Test
{
    public class xUnitAttributesAssertCheetsheet
    {
        /// <summary>
        /// xunit.assert's version is  2.4.1
        /// </summary>
        [Fact]
        public void AllAsserts()
        {

            #region Assert.Equal(object expected,object actual)
            //Comparing Integer values
            //Result: Pass
            Assert.Equal(1, 1);

            //Comparing String values
            Assert.Equal("TechnoThirsty", "TechnoThirsty");
            Assert.Equal("TechnoThirsty", "technoThirsty", ignoreCase: true);

            Assert.Matches("[A-Z]{1}[a-z]+ [A-Z]{1}[a-z]+", "Techno Thirsty");

            //Comparing double value for 1 precision value
            Assert.Equal(1.95, 1.99, 1);

            //compare List 
            Assert.Equal(new List<String> { "techno", "thirsty" },
                         new List<String> { "TECHNO", "THIRSTY" },
                         StringComparer.CurrentCultureIgnoreCase);


            #endregion Assert.Equal


            #region Assert.NotEqual
            Assert.NotEqual("1", "2");
            Assert.NotEqual(1, 2);
            Assert.NotEqual(new List<String> { "A", "B" },
                            new List<String> { "1", "2" });

            #endregion Assert.Equal


            #region Asset.Contains(expected,actual)

            Assert.Contains("Techno",
                               "TechnoThirsty",
                               StringComparison.CurrentCultureIgnoreCase);

            Assert.Contains("Techno", new List<String> { "Techno", "Thirsty" });
            Assert.Contains("Techno",
                            new List<String> { "techno", "Thirsty" },
                            StringComparer.CurrentCultureIgnoreCase);

            #endregion Asset.Contains(expected,actual)

            Assert.DoesNotContain("Techno", new List<String> { "Technos", "Thirsty" });

            List<string> lst = new List<String> { "Technos", "Thirsty" };
            //iterate through all values and check for null,whitespace or blank 
            Assert.All(lst, d => Assert.False(string.IsNullOrWhiteSpace(d)));

            Assert.Empty(new List<String>());
            Assert.NotEmpty(new List<String> { "Techno", "Thirsty" });

            Assert.True(true);
            Assert.False(false);
            Assert.Null(null);
            Assert.NotNull(new String("TechnoThirsty"));

            Assert.InRange(1, 0, 10);
            Assert.NotInRange(-1, 0, 10);

            Assert.IsType(Type.GetType("System.Int32"), 1); // will not return value 
            Assert.IsType<Int32>(1); // will return value 
            Assert.IsNotType(Type.GetType("System.Double"), 1);
            Assert.IsNotType<Double>(1);

            Assert.IsAssignableFrom<int>(111); // will return value
            Assert.IsAssignableFrom(Type.GetType("System.Int32"), 1);


            Assert.ThrowsAsync<Exception>(() =>
                        {
                            //Method which we want to call
                            throw new Exception();
                        });
            Assert.ThrowsAsync<ArgumentException>(() => GetDetails(""));
            Assert.ThrowsAsync<ArgumentException>("name", () => GetDetails(""));


            var obj1 = new object();
            var obj2 = new object();

            Assert.Same(obj1, obj1);
            Assert.NotSame(obj1, obj2);
        }

        public async Task<int> GetDetails(string s)
        {
            if (string.IsNullOrWhiteSpace(s))
                throw new ArgumentException(nameof(s), "");

            return 0;

        }

        [Fact]
        [Trait("Category", "TraitDemo")]
        public void TraitDemo()
        {

        }

        /// <summary>
        /// We can use InlineData Instead of repeated code
        /// </summary>
        /// <param name="input"></param>
        /// <param name="expected"></param>
        [Theory]
        [InlineData(1, 1)]
        [InlineData(2, 2)]
        [InlineData(333, 333)]
        public void DataDrivenTestByAddingInlineData(int input, int expected)
        {
            Assert.Equal(expected, input);
        }


        [Theory]
        [ClassData(typeof(TestDataGeneratorForClassData))]
        public void DataDrivenTestByClassData(int input, int expected)
        {
            Assert.Equal(expected, input);
        }

        [Theory]
        [MemberData(nameof(InternalClassTestData.TestData), MemberType = typeof(InternalClassTestData))]
        public void DataDrivenTestByMemberDataToShareInDifferentTestClasses(int input, int expected)
        {
            Assert.Equal(expected, input);
        }



        [Theory]
        [MemberData(nameof(ExternalClassForTestData.TestData),
            MemberType = typeof(ExternalClassForTestData))]
        public void DataDrivenTestByClass(int input, int expected)
        {
            Assert.Equal(expected, input);

        }

        [Theory]
        [ExternalClassForTestDataWithDataAttribute]
        public void DataDrivenTestByClassWithAttribute(int input, int expected)
        {
            Assert.Equal(expected, input);

        }

        [Fact(Skip = "Add your reason for skipping test")]
        public void SkippableTest()
        {
            //AAA
        }

    }

    public class TestDataGeneratorForClassData : IEnumerable<object[]>
    {
        private readonly List<object[]> _data = new List<object[]>
        {
            new object[] {1,1},
            new object[] {101,101}
        };

        public IEnumerable<object[]> TestData => _data;

        public IEnumerator<object[]> GetEnumerator() => _data.GetEnumerator();

        IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
    }

    public class InternalClassTestData
    {
        public static IEnumerable<object[]> TestData
        {
            get
            {
                yield return new object[] { 100, 100 };
                yield return new object[] { 1, 1 };
                yield return new object[] { 50, 50 };
                yield return new object[] { 75, 75 };
                yield return new object[] { 101, 101 };
            }
        }
    }

    //Without Data Attribute
    public class ExternalClassForTestData
    {
        public static IEnumerable<object[]> TestData
        {
            get
            {
                string[] csvLines = File.ReadAllLines("TestData.csv");

                var testCases = new List<object[]>();

                foreach (var csvLine in csvLines)
                {
                    IEnumerable<int> values = csvLine.Split(',').Select(int.Parse);

                    object[] testCase = values.Cast<object>().ToArray();

                    testCases.Add(testCase);
                }

                return testCases;
            }
        }
    }

    //With Data Attribute
    public class ExternalClassForTestDataWithDataAttribute : DataAttribute
    {
        public override IEnumerable<object[]> GetData(MethodInfo testMethod)
        {
            string[] csvLines = File.ReadAllLines("TestData.csv");

            var testCases = new List<object[]>();

            foreach (var csvLine in csvLines)
            {
                IEnumerable<int> values = csvLine.Split(',').Select(int.Parse);

                object[] testCase = values.Cast<object>().ToArray();

                testCases.Add(testCase);
            }

            return testCases;
        }
    }

}
```
