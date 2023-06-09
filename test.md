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

