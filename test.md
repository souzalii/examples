### Model solution

Determining whether our tests achieve some level of logic coverage (Restricted Active
Clause Coverage would be a sensible choice) *would* be a reasonable use of
time.

To arrive at that conclusion, we must estimate the effort involved and what value we
would gain from determining the level of coverage. For systems which contain
many multi-clause predicates, where the tolerance for risk is low, and where we may
not have already achieved coverage through other means, it probably *is*
a good idea to assess logic coverage.

But is this the case here? A reading of the specification plus reasonable
guesses as to what the implemented code will look like suggest
most predicates are likely to be simple ones consisting of a single clause.
Some might argue that looking at, say, the `SegmentSubcommand` constructor and
other constructors, the condition for throwing a `SemanticError` is somewhat complex.
(We need to determine that the parameters *are* syntactically valid,
and then must check whether any of the conditions for throwing a semantic error are
met.) That is true, when we lack implemented code and can work only
off the design and/or specification documents.
But when implemented in Java code, these apparently
complex conditions will just be checked by code which goes through each parameter one by
one, and will likely have a only one or two clauses per "`if`" statement. (We assume
here that since we already have a *partial* implementation, by the time a test suite
is done, we will have a complete or almost complete implementation.)

We might be uncertain about the code in a parser like the `CommandParser` class, but
if it uses existing Java parser libraries (rather than us implementing a parser
library ourselves), we could assume the code in it
will be typical of most Java code, and have at most a couple of clauses per
"`if`" statement.

So, most code predicates are likely simple, therefore we probably would have
achieved RACC already for those, through ISP and perhaps looking at
branch/decision coverage reports
from a tool like JaCoCo. This means the effort involved to determine whether RACC is
met for other predicates is not too high; we just need to look for cases where there are
"`if`" statements with more than one clause, since those wouldn't
necessarily achieve RACC through other means.

It is difficult to put a figure on this, but we might estimate that the work involved
as being a less than a person-week's worth of effort (but maybe more than a day, to
do a thorough job and report on findings). What value would we gain? We would have
more confidence that the complex portions of code we identified would work correctly.

Therefore, unless there are other pressing demands on the team's time, I *would*
recommend we check whether RACC is met. It is true that our team's product is only
a prototype, but still, it would be a setback to the team if (for instance) the
prototype failed while being trialled and demonstrated, when we could have
removed errors through testing while implementation was still being done.

*[An answer could also reasonably conclude we **shouldn't** spend time
on assessing logic coverage, since the stakes are low (this is a prototype)
and on other assumptions we might decide the codebase is unlikely to contain
many predicates that wouldn't be thoroughly tested through other means.]*


**Model answer**

The method documentation for `CommandParser.parse` states that on success, a `Command` object is constructed. However, it isn't returned (the return type is `void`), so
presumably, each class implementing the `CommandParser` interface would specify where this object is to be constructed (e.g. perhaps in a private instance variable `result`,
accessible via a `getResult()` method). Since we're asked to write tests based just on this interface, it follows
our tests can only observe whether an exception was thrown or not -- we can't obtain and analyse the constructed `Command` object.

We are asked to derive tests which have production coverage. We aren't asked to write tests for the case where an invalid command is passed to the `parse()`
method, so we assume testing valid commands is sufficient.

So, our tests (implemented in JUnit, with the class under test being `CommandParserImpl`) will all look like this:

```java
  @Test
  public void shopFlightFare_parses_001() {
    String commandString = "shop flight fares AAA BBB OneWay P 2023-01-01";
    CommandParserImpl cp = new CommandParser();
    // shouldn't throw an exception
    cp.parse(commandString);
  }
```

As test cases, our tests would be of the form:

<div style="margin-left: 3em;">
<table>
<thead>
<tr class="header">
<th style="text-align: center;"><strong>Test case ID</strong></th>
<th style="text-align: center;"><strong>Test values</strong></th>
<th style="text-align: center;"><strong>Expected result</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;">001</td>
<td style="text-align: left;"><code>"shop flight fares AAA BBB OneWay P 2023-01-01a"</code></td>
<td style="text-align: left;">No exception thrown</td>
</tr>
</tbody>
</table>
</div>


Since they'll all be of that form, we can just identify them from here on by giving
the test value (i.e. the array of lines being passed in; we'll assume newline
characters have been stripped from the ends).

&nbsp;\

So, we need to make sure we cover all the following productions:

<img src="https://images2.imgbox.com/1e/ae/sOlJx5IK_o.jpg" alt="table" width="700" height="315" style="margin: 2rem;" />


&nbsp;\

Naively, we might think this requires 50 tests (the sum of the number of productions), but we can actually
get by with fewer, since some tests will exercise multiple productions.

For instance, the following test --

```
  {"shop flight fares ABC DDD OneWay P 0123-45-67"}
```

exercises 4 of the `<a_to_d>` productions, one of the `<cabin_type>` productions, and 8 of the `<digit>`
productions. We could add a similar test which includes the digits `8` and `9`, and we'd have covered
the `<a_to_d>` and `<digit>` productions (running total: 2 tests). To cover off all the `<zero_to_twenty>`
productions, we could make this second test be a return trip, with a trip
length of 0 days, and add two more tests with trip length of 10 and 20 days, respectively.
So that is 4 tests total, which cover the `<a_to_d>`, `<trip_length>`, `<airport_code>`, `<digit>`
and `<trip>` productions.

&nbsp;\

Because an "air book req" command includes multiple segment subcommands, we could write a test with
(say) six subcommands in it, each with a different `<cabin_type>`, and we'd have covered the `<cabin_type>`
productions. If the flight numbers contained all of the letters "I", "J", "K" and "L", we'd have
covered the `<flight_number>`, `<airline_code>` and `<up_to_4_digits>` rules. If we added 4 more
segment subcommands so we have a total of 10, then we could cover each of the 10 `<num_people>`
productions. We would also have covered the `<eoc>`, `<segment_subcommand>`, `<air_book_req>`
and `<gladius_command>` productions.

So that means we can write 5 tests which have production coverage. (We have not made particular
efforts to minimize the number of tests; it could be done with slightly fewer.)

&nbsp;\

***A listing of the tests is not necessary; they've now been described
sufficiently. But for completeness, a listing follows.***

A list of plausible test cases, then (identified just by their input string; we show multiple
lines in one string), is:

&nbsp;\

- `shop flight fares ABC DDD OneWay P 0123-45-67`
- `shop flight fares ABC DDD Return 0 P 8999-99-99`
- `shop flight fares ABC DDD Return 10 P 8999-99-99`
- `shop flight fares ABC DDD Return 20 P 8999-99-99`
- ```
  air book req
  seg AAA AAA II000 000-00-00 P 1
  seg AAA AAA JJ000 000-00-00 F 2
  seg AAA AAA KK000 000-00-00 J 3
  seg AAA AAA LL000 000-00-00 C 4
  seg AAA AAA LL000 000-00-00 S 5
  seg AAA AAA LL000 000-00-00 Y 6
  seg AAA AAA LL000 000-00-00 Y 7
  seg AAA AAA LL000 000-00-00 Y 8
  seg AAA AAA LL000 000-00-00 Y 9
  seg AAA AAA LL000 000-00-00 Y 10
  EOC
  ```
  Solution
Reference solution. Students should have solutions that look roughly like this.

  import static org.junit.jupiter.api.Assertions.*;
  import java.time.LocalDate;
  import org.junit.jupiter.api.Test;

  class SegmentSubcommandTest {

    /* TESTS IMPLEMENTED WILL DEPEND ON ANSWER TO QUESTION 6.
     * However, some typical tests are shown here.
     * Full answers will include Javadoc documentation identifying
     * which test case they are implementing.
     */


    /** Test the "base case" (no exceptions thrown). For completeness,
     * we could also use the constructed object's accessor methods
     * to test the attributes were properly set.
     */
    @Test
    void testSegmentSubcommandBaseCase() {
      String origin = "PER";
      String destination = "SYD";
      String flightNumber = "QF642";
      LocalDate departureDate = LocalDate.now().plusDays(1);
      CabinType cabinType = CabinType.EconomyClass;
      int numPeople = 1; 

      SegmentSubcommand ss = 
          new SegmentSubcommand(origin, destination, flightNumber, 
                                departureDate, cabinType, numPeople);

      // using the `doesNotThrow` assertion would be better
      // style, but this is ok too 
      assertTrue(true, "no exception thrown");
    }


    /** Test the case where the origin is syntactically
     * invalid (consists of an empty string) - a SyntacticError
     * should be thrown.
     */
    @Test
    void testSegmentSubcommandInvalidOrigin() {
      String origin = "";
      String destination = "SYD";
      String flightNumber = "QF642";
      LocalDate departureDate = LocalDate.of(2025, 1, 1);
      CabinType cabinType = CabinType.EconomyClass;
      int numPeople = 1; 

      @SuppressWarnings("unused")
      SyntacticError thrown = assertThrows(
          SyntacticError.class,
          () -> new SegmentSubcommand(origin, destination, flightNumber, 
                                      departureDate, cabinType, numPeople),
          "Expected SegmentSubcommand() to throw due to invalid origin \"\"."
      );
    }


     /** Test the case where the origin and destination are the
      * same - a SemanticError should be thrown.
      */
    @Test
    void testSegmentSubcommandSameOriginAndDest() {
      String origin = "PER";
      String destination = "PER";
      String flightNumber = "QF642";
      LocalDate departureDate = LocalDate.of(2025, 1, 1);
      CabinType cabinType = CabinType.EconomyClass;
      int numPeople = 1; 

      @SuppressWarnings("unused")
      SemanticError thrown = assertThrows(
          SemanticError.class,
          () -> new SegmentSubcommand(origin, destination, flightNumber, 
                                      departureDate, cabinType, numPeople),
          "Expected SegmentSubcommand() to throw due to origin == destination."
      );
    }
  }
  
  
  
  **Model answer**

To perform input space partitioning, we take the following steps:

#### 1. Identify functions

Here, the function being tested is the `SegmentSubcommand` constructor.
It has a number of explicit parameters (identified in the next step),
as well as several implicit ones. As a Java constructor, it does not
have a "return type": rather, it's responsible for ensuring that
newly allocated memory for a `SegmentSubcommand` object is properly
initialized. However, when modelling it as a function, it is easiest
to make the assumption that a `SegmentSubcommand` object is returned.

#### 2. Identify parameters

The explicit parameters to the constructor, their types, and a brief description are as follows:

- `origin: String` - an airport code
- `destination: String` - an airport code
- `flightNumber: String` - a flight number (an airline code and 1-4 digits)
- `departureDate: LocalDate` - the departure date
- `cabinType: CabinType` - an enum representing the seat class (economy, first class, etc)
- `numPeople: int` - number of seats to book

These parameters come with some preconditions: when testing, we needn't consider
the case where these are violated (and in fact, writing tests for such cases is impossible).
Specifically, it's an implied precondition that none of the parameters can be `null`.
(And `numPeople`, as a primitive `int` type, can't be `null` anyway.)

There is also an implicit parameter. We are told that in the Gladius system,
semantic validation of airport codes and flight numbers is done by consulting a database;
therefore, the state of the database is an implicit parameter.
We aren't told what the preconditions are for this database, but we can reasonably assume
as a precondition that the database is not corrupt/in an erroneous state. Should we
assume the database can be connected to? Probably that *is* worth writing tests for;
but since we aren't given any information about what happens if we can't connect, for the
purposes of this question we'll discount that possibility.

#### 3. Model characteristics of the parameters

We will not exhaustively list all characteristics, but from the constructor
documentation, we can identify the following:

- For each of `origin`, `destination`, and `flightNumber`:
  - Is it syntactically valid, yes or no?
  - Assuming it is syntactically valid: is it semantically valid, yes or no?

  (6 characteristics in total; let us call these characteristics Origin<sub>synt</sub>,
  Origin<sub>sem</sub>, and so on.) 
- Considering `origin` and `destination` together:
  - Are they the same, yes or no? (If they are, an exception gets thrown.)
- `departureDate`:
  - Is it after today's date or not?
- `numPeople`:
  - Is it: (a) less than 1, (b) between 1 and 10, inclusive, or (c) greater than 10?\
    (Partitions (a) and (c) result in an exception being thrown.)  

That is a total of 9 characteristics; for the purposes of this question, we'll mostly limit
ourselves to those. For writing tests, we'd also want:

- `cabinType`:
  - Give each cabin type its own partition.

#### 4. Choose partitions, and values from within them.

We're asked to achieve Base Choice Coverage; for this purpose,
we want to identify, for each partition, a "base choice"
(usually a "normal" or "common" case). It seems reasonable for
the base choice to be one where an exception is *not* thrown.

So our choices are:

- `origin`, `destination` and `flightNumber` should all be both
  syntactically and semantically valid
- `origin` and `destination` should be different
- `departureDate` should be after today's date
- `numPeople` should be within 1 and 10, inclusive.
- `cabinType` - any would do, we select `EconomyClass`.

What values from each should we choose? One consideration is that
choosing a value for `departureDate` is tricky if we want our tests
to be completely deterministic (i.e., have the exact same inputs and outputs
every time). We would need to "mock" Java's idea of "what the current date is".
There are several ways of doing this (by, e.g. using mocking libraries, or
running the tests in a Docker container where the date can be simulated),
but for simplicity in this question, we will call `LocalDate.now()` to
get the current date (at the time the test is run), and call `plus(1)` on it to
get tomorrow's date. Other than that, we could use the following values:

- `origin: "PER"`
- `destination: "SYD"`
- `flightNumber: "QF642"`
- `numPeople: 1`
- `cabinType: EconomyClass`

To achieve base choice coverage, we would now also go through each characteristic,
and select test values for the *non*-base partitions. For this question, however,
we are asked only to come up with three test cases, so we will limit ourselves to
the following 2 non-base partitions:

- For a non-syntactically-valid `origin` (non-base partition from the Origin<sub>syn</sub>
  characteristic), we could select any string; the empty
  string (`""`) is certainly not syntactically valid.
- For a non-base choice of "Are `origin` and `destination` the same?", we would choose
  "yes".

We then end up with the following 3 test cases:

|       Test ID       |                        Test summary                        |                                    Input values                                    |     Expected output     |
|:--------------------|:-----------------------------------------------------------|:-----------------------------------------------------------------------------------|:------------------------|
| 001-base            | Base case                                                  | Origin = "PER", destination = "SYD", flight number = "QF642", number of people = 1 |                         |
| 002-origin-syn      | As for base case, but with a syntactically invalid origin. | As for base case, but origin = ""                                                  | Throws `SyntacticError` |
| 003-origin-dest-sem | As for base case, but origin and destination are the same. | As for base case, but destination = "PER"                                          | Throws `SemanticError`  |


**Model answer**

There are very few preconditions for the correct operation of the `ShopFlightFare` constructor:

- In general, it's an implied precondition of any Java method that any of its parameters
  which are of reference type must not be null. Thus, we can assume that
  `origin`, `destination`, `tripType`, `cabinType` and `departureDate`
  must not be null.
- However, `lengthOfStay` is explicitly *permitted* to be null, as long
  as `tripType` is `ONE_WAY`. So the precondition here is that
  if `tripType` is `RETURN`, `lengthOfStay` must not be null.
- The Javadoc documentation for `ShopFlightFare` does not explicitly mention
  any other preconditions. However, we know from the project specification
  that things like IATA codes, currency codes and airline codes are
  looked up in a database (or using some similar service).

  So it could therefore be a precondition that we have valid databases
  containing this information, as well as a connection to those databases.
  (It need not be the case that the connection be stored as an instance variable
  in `ShopFlightFare`; our code might obtain it from e.g. an entirely different
  `IataDatabase` class.)

As long as those (fairly minimal) preconditions are satisfied, the
`ShopFlightFare` constructor guarantees us a result: *either* the
construction of a valid `ShopFlightFare` object, or that one of
two types of exception will be thrown (`SyntacticError` or `SemanticError`).

To be precise, the postconditions are that:

- a `SyntacticError`
  exception will be thrown if and only if the `origin` or `destination`
  aren't syntactically valid IATA codes (as defined in the project spec),
  or the `flightNumber` isn't a syntactically valid flight number
  (again, as defined in the spec; 2 alphanumeric characters followed
  by 1 to 4 digits).
- a `SemanticError` exception will be thrown if and only if at least
  one of the following are true (and assuming some syntactic
  invalidity hasn't already caused a `SyntacticError` to be thrown).

  - the origin is the same as the destination
  - the origin or destination are not semantically valid (i.e. cannot be
    found in a database of IATA codes)
  - the flight number isn't semantically valid (e.g. because the airline code
    portion can't be found in a databse of airline codes. It isn't clear from the
    spec, but presumably flight numbers could be semantically invalid on their
    own, as well -- for instance, because the flight number specified doesn't
    actually go from the origin to the destination).
- in all other cases, a valid `ShopFlightFare` object is constructed.



Criterion 1
Does the submitted answer manage to identify relevant facts from the scenario and exclude any irrelevant facts? Does it correctly identify what topics covered in the unit are irrelevant, and avoid discussion of any that are irrelevant?

You should rate the submitted answer based on how well it achieves the criterion above. The material below provides guidance on what topics and facts are relevant or irrelevant.

Completing a peer review
Note that there are two criteria you need to select an option for, before you can save your answer: "relevance" (this criterion) and "justification".

You should also provide a comment, in the box labelled "Overall feedback". There is no specific format required for this, but you might like to include a list of "Things that were done well" and "Things to improve".

Relevant facts/topics:
If a submission misses one or more of these top-level dot points, that's a major omission. Material in parentheses or secondary dot points might be omitted or only briefly covered, however, and that's fine.

Both requirements are examples of non-functional requirements. The first is an example of a security requirement, the second an example of a performance requirement. (Functional requirements describe what a system does; non-functional requirements describe the way the system does something (e.g. securely, performantly, etc.)

A submitted answer might cite a (brief) definition of a non-functional requirement. For instance, Pressman (9th ed, sec 7.2.5) describes a non-functional requirement as follows: "A nonfunctional requirement (NFR) can be described as a quality attribute, a performance attribute, a security attribute, or a general constraint on a system".
 

Security requirements can be tested using security testing. The answer should briefly explain what is involved in security testing, based on material from the textbooks or other sources.

For instance: Security testing involves the use of techniques like dynamic analysis, fuzz testing, and penetration testing (Pressman, 9th ed, sec. 21.7 and 18.2).

 

Performance requirements can be tested using performance testing.

The submission might give a reference (e.g. Pressman, 9th ed, sec 21.8), and/or mention that performance testing is often coupled with stress testing.
 

The answer should briefly say something about how performance tests are done.

For instance: performance tests involve simulating real-world loads on a system.

 

Input space partitioning (ISP) models the subject under test as a mathematical function, which consumes inputs and emits outputs. (It divides up the domain of the function into partitions, which are used to derive test cases for the subject under test.)
 

Optional relevant facts/topics:
A student may discuss this, but might also leave it as implicit:

ISP requires precisely specified inputs and outputs for a function, including clear preconditions and postconditions.
Irrelevant facts and topics
In general, an answer that contains more than around 2 medium-length paragraphs is likely to be giving unnecessary and irrelevant detail.

Note that the question does not ask students to

carry out the steps involved in ISP
outline the steps of ISP
suggest characteristics or partitions
repeat the definition of ISP (or the definitions of function, domain, codomain, partition, etc.)
identify functions of the system that could be used for ISP
suggest how the proposed system might be designed, or speculate as to design details
propose or list test cases
provide Java code
so answers that do any of those things (unless it's very brief) contain unnecessary detail. You should decide whether this is a minor issue (a little irrelevant detail) or a major issue (significant amounts of irrelevant detail).

The question also does not ask for students to

explain, in extensive detail, how the requirements can be improved (though a brief mention of problems is fine)
explain, in extensive detail, how security or performance requirements are tested (though a brief mention is fine)
Discussion of other topics covered in class (graph testing, mocks, logic-based testing, test frameworks) is also irrelevant to this question.





No answer given, or answer is not comprehensible, or doesn't answer the question.

Not yet satisfactory – little or no evidence of ability to identify relevant facts/topics.

Satisfactory – some relevant facts and topics identified but some major omissions

Proficient – all (or nearly all) relevant facts/topics identified, with at most minor omissions/errors
Criterion 2
Criterion 2
Does the submission provide a clear answer to the question posed, and a good justification for that answer?

You should rate the submitted answer based on how well it achieves the criterion above. The material below provides guidance on what constitutes a good justification in this case.

Sensible conclusion
The submission need not have a distinct section titled "conclusion", but it must, somewhere, provide a clear answer to the question posed ("Does ISP provide a good way to devise tests based on these two requirements?") – possible conclusions include "Yes", or "No", or "Partly yes". Failure to provide a clear conclusion is a significant omission.

In general, any answer that concludes that ISP should be used (solely) to devise tests for the requirements is flawed in its reasoning. ISP requires that a system (or parts of it) be modelled as a function with clearly specified inputs and outputs.

The non-functional requirements given (a security requirement and a performance requirement) can't be straightforwardly modelled in such a way, and in any case, there are more appropriate ways of testing those requirements (namely, with security testing of various sorts, and performance testing). Attempting to use ISP, instead of those appropriate techniques, will almost certainly result in (a) far more work than is necessary, and/or (b) major gaps in the testing.

However, the submission may conclude that ISP can contribute in a small way to security testing or performance testing – for instance, it might be used to identify edge or boundary cases where security flaws might lurk, or be used to identify what "typical" requests would look like for the purpose of performance testing.

Appropriate justifications for conclusion
In addition to a sensible conclusion, the submitted answer must provide logical reasons for that conclusion.

Some possible reasons are given below.

Against ISP being applicable:

The ISP technique can derive test cases based on the domain when we model a system as a function; but the sorts of tests required for performance and security testing don't test what the system does (it's functionality), but how it does it.

The ISP technique partitions up the domain of a function to help in constructing test cases; but what's needed in performance testing (for instance), is not a partitioning up of a functional domain, but the ability to simulate many requests being supplied to the system's servers at once. Whether the requests are valid or invalid is less relevant than how many are being made at once.

Security testing techniques (for instance, the ones suggested by Pressman) don't depend on a partitioning up of the input domain, but instead, on particular ways of analysing and simulating threats to the system.

Even if ISP could be sensibly applied to the security requirement, that requirement is currently too vague – it needs to be made more precise before it is a testable requirement.

 

In favour of ISP being (partially) applicable:

Although the ISP technique does not directly result in tests that could be used to test performance or security, it could still be used to help derive part of the test cases used to test these attributes. For instance, we could generate a typical "request" to the system using ISP, and use this as part of performance testing by simulating the effect of many such requests all being submitted to the system at the same time.

The ISP technique might also assist in identifying test cases which reveal security flaws, since there is a focus on generating partitions (and test values) which represent "unusual" and "border" cases, which could reveal security flaws.

 

Submitted answers may come up with other justifications for or against ISP being applicable.

Poor justifications for conclusion/poor reasoning
If an answer simply tries to apply ISP to the requirements, despite the fact that it's a very poor fit – that counts as a major omission or error.

Answers that are self-contradictory (for instance – answers which say that ISP is inappropriate, but then apply it anyway) should be considered to contain a major error.


Criterion 2
Does the submission provide a clear answer to the question posed, and a good justification for that answer?

You should rate the submitted answer based on how well it achieves the criterion above. The material below provides guidance on what constitutes a good justification in this case.

Sensible conclusion
The submission need not have a distinct section titled "conclusion", but it must, somewhere, provide a clear answer to the question posed ("Does ISP provide a good way to devise tests based on these two requirements?") – possible conclusions include "Yes", or "No", or "Partly yes". Failure to provide a clear conclusion is a significant omission.

In general, any answer that concludes that ISP should be used (solely) to devise tests for the requirements is flawed in its reasoning. ISP requires that a system (or parts of it) be modelled as a function with clearly specified inputs and outputs.

The non-functional requirements given (a security requirement and a performance requirement) can't be straightforwardly modelled in such a way, and in any case, there are more appropriate ways of testing those requirements (namely, with security testing of various sorts, and performance testing). Attempting to use ISP, instead of those appropriate techniques, will almost certainly result in (a) far more work than is necessary, and/or (b) major gaps in the testing.

However, the submission may conclude that ISP can contribute in a small way to security testing or performance testing – for instance, it might be used to identify edge or boundary cases where security flaws might lurk, or be used to identify what "typical" requests would look like for the purpose of performance testing.

Appropriate justifications for conclusion
In addition to a sensible conclusion, the submitted answer must provide logical reasons for that conclusion.

Some possible reasons are given below.

Against ISP being applicable:

The ISP technique can derive test cases based on the domain when we model a system as a function; but the sorts of tests required for performance and security testing don't test what the system does (it's functionality), but how it does it.

The ISP technique partitions up the domain of a function to help in constructing test cases; but what's needed in performance testing (for instance), is not a partitioning up of a functional domain, but the ability to simulate many requests being supplied to the system's servers at once. Whether the requests are valid or invalid is less relevant than how many are being made at once.

Security testing techniques (for instance, the ones suggested by Pressman) don't depend on a partitioning up of the input domain, but instead, on particular ways of analysing and simulating threats to the system.

Even if ISP could be sensibly applied to the security requirement, that requirement is currently too vague – it needs to be made more precise before it is a testable requirement.

 

In favour of ISP being (partially) applicable:

Although the ISP technique does not directly result in tests that could be used to test performance or security, it could still be used to help derive part of the test cases used to test these attributes. For instance, we could generate a typical "request" to the system using ISP, and use this as part of performance testing by simulating the effect of many such requests all being submitted to the system at the same time.

The ISP technique might also assist in identifying test cases which reveal security flaws, since there is a focus on generating partitions (and test values) which represent "unusual" and "border" cases, which could reveal security flaws.

 

Submitted answers may come up with other justifications for or against ISP being applicable.

Poor justifications for conclusion/poor reasoning
If an answer simply tries to apply ISP to the requirements, despite the fact that it's a very poor fit – that counts as a major omission or error.

Answers that are self-contradictory (for instance – answers which say that ISP is inappropriate, but then apply it anyway) should be considered to contain a major error.




You are writing tests for a method – call it method X – which takes one parameter, an int. Allowable values for the int are from 0 to 6, inclusive (and it's a precondition of method X that the value be an allowed one). Which of the following would count as a valid partitioning of the domain of method X?
The correct answer is: non-zero even allowable numbers ({2,4,6}), odd allowable numbers ({1,3,5}), and zero ({0}).

You are writing a class, SalaryBonusApplier, which makes a request over the network to your company's HR database. During normal operation, your class makes use of a DatabaseConnection object to communicate with the HR database. However, during testing, you will be using an in-memory database instead of a real database, and you wish to verify that that the correct requests are being made to it. Which of the following would be most appropriate to use here?  spy, fake object
  
  
## Suppose you are creating tests for a MathOperations class, which contains a method safeAdd() with the following signature and documentation:


  /** If the sum of <code>a</code> and <code>b</code>
    * lies within the representable range of an <code>int</code>,
    * returns the sum; but if not, throws an IllegalArgumentException.
    */
  static int safeAdd(int a, int  b)


Each the following options gives sets of test values that could be passed to the safeAdd() method as part of a JUnit test (each (a,b) pair constitutes one test case). Which of them would be most suitable for thoroughly testing the safeAdd() method?
The correct answer is: (a = 0, b = 0), (a = 1, b=-1), (a = -1, b = -1), (a = 1, b=1), (a = 2147483647, b = 1), (a = -2147483648, b = -1), (a = 2147483647, b = -2147483648)
