# SQL TUNING
This course is to help use tune (increase efficiency) of our SQL queries. the solution provided from the methods bellow wont work in all the SQL databases out there but each and every one will work on atleast one of the main type Databases.

# Simple Searches | General tuning
## Code for Points
we can create a code for points system to improve perfomance. Each search condition component has a "point count"—the better the component, the higher the score. You can see from the allotted points below.

### Search Condition Point Counts for Operators
| Operator | Points |
|----------|--------|
| =        | 10     |
| >        | 5      |
| >=       | 5      |
| <        | 5      |
| <=       | 5      |
| LIKE     | 3      |
| <>       | 0      |

<br>

### Search Condition Point Counts for Operands
| Operator                | Points |
|-------------------------|--------|
| Literal alone           | 10     |
| Column alone            | 5      |
| Parameter alone         | 5      |
| Multioperand expression | 3      |
| Exact numeric data type | 2      |
| Other numeric data type | 1      |
| Temporal data type      | 1      |
| Character data type 0   | 0      |
| NULL                    | 0      |


<br>

### example 1
```sql
... WHERE smallint_column = 12345
```

This example gets a total of 27 points, calculated as follows:
- Five points for the column (smallint_column) alone on the left
- Two points for the exact numeric (smallint_column) operand data type
- Ten points for the equals operator
- Ten points for the literal (12345) alone on the right

<br>

### example 2
The point count for this type of search condition is much lower—only 13:
- Five points for the column (char_column) alone on the left
- Zero points for the CHAR (char_column) operand data type
- Five points for the greater than or equal to operator
- Three points for the multioperand expression (varchar_column || 'x') on the right
- Zero points for the VARCHAR (varchar_column) operand data type

## Constant Propagation
Formally, the Law of <b>Transitivity</b> states that:

```sql
IF
(A <comparison operator> B) IS TRUE AND (B <comparisonoperator> C) IS TRUE
THEN
(A <comparison operator> C) IS TRUE AND NOT (A <comparisonoperator> C) IS FALSE
```
This means we only need to compare `A` and `C` and we can have just 1 conditional statement instead of 2.

(Comparison operator is any one of: `=` or `>` or `>=` or `<` or `<=` but not one of: `<>` or `LIKE`.)

### example 1
```sql
... WHERE column1 < column2
      AND column2 = column3
      AND column1 = 5
```

this can be simplified to
```sql
... WHERE 5 < column2
      AND column2 = column3
      AND column1 = 5
```

By using `5` instead of `column1` yo get Ten points for the literal (5) alone.

## Dead Code Elimination 

This means all queries with unused code shold be used 

## Ensure You Use the Right DBMS
There are several ways to ensure that a specific DBMS (and no other) executes an expression. Here are three examples, all of which use nonstandard SQL extensions:
```sql
... WHERE :variable = 'Oracle'
      AND /* Oracle-specific code here */
```

## Constant Folding
`1+1-1-1` is equal to `0`. i sue this example since in C `x=1+1-1-1` is automatically folded to `x=0` and not computed. But SQL cannot fold the following.

```sql
... WHERE column1 + 0

... WHERE 5 + 0.0

... WHERE column1 IN (1, 3, 3)

... CAST(1 AS INTEGER)

... WHERE 'a' || 'b'

... WHERE a - 3 = 5

```

these can quickly be folded to

```sql
... WHERE column1 /* any number plus zero is that number */

... WHERE 5 /* any number plus zero is that number */

... WHERE column1 IN (1, 3) /* you only need one three */

... /* this can be completely removed */

... /* this can be completely removed */

... WHERE a = 8  

```

## Case-Insensitive Searches
A slightly faster search assumes that the data is clean and asks for the only reasonable combinations, like this:

```sql 
... WHERE column1 = 'SMITH'
       OR column1 = 'Smith'
```

than this

```sql 
... WHERE UPPER(column1) = 'SMITH'
```

You can also take advantage of dead code elimination so that the 'Smith'
search happens only when the DBMS is case sensitive. Here's how

```sql
... WHERE column1 = 'SMITH'
      OR  ('SMITH' <> 'Smith' AND column1 = 'Smith')
```
This will work fast for any database system that is not case sensitive or set to be case sensitive.

## Sargability
The left side of a search condition should be a simple column name; the right side should be an easy- to-look-up value.

To enforce this rule, all DBMSs will transpose the expression:
```sql
5 = column1
```

to:

```sql
column1 = 5
```

When there's arithmetic involved, though, only some DBMSs will transpose. For example, we tested this transform:

```sql
... WHERE column1 - 3 = -column2
```

transforms to:

```sql
... WHERE column1 = -column2 + 3
```

On a 32-bit computer, arithmetic is fastest if all operands are INTEGERs (because INTEGERs are 32- bit signed numbers) rather than SMALLINTs, DECIMALs, or FLOATs. Thus this condition:

```sql
... WHERE decimal_column * float_column
```
is slower than:

```sql
... WHERE integer_column * integer_column
```

## Summary
To sum up the above point. the following has to be true.
- The left side of a search condition should be a simple column name; the right side should be an easy- to-look-up value.
- Each component of a search condition has a point count. The higher the points, the faster the component.  The condition that takes the least time gets the most points.
- Put multiple expressions in the correct order.
- Use the Law of Transitivity and the concept of constant propagation to substitute a literal for a column name or column expression whenever you can do so without changing the meaning of an expression.
- Some DBMSs won't fold most obvious-looking (to a C programmer) expressions. Don't use this principle as the reason to always transform such expressions when you find them in old code though—usually they're there for historical reasons. Remember Rule #1: Understand the code before changing it.
- If the code involves an obvious math expression, do evaluate it and transform the condition to the evaluated result. Constant folding can lead to constant propagation and is therefore A Good Thing.
Avoid functions on columns.
- If you can't avoid functions, don't use UPPER to ensure case insensitivity. Use LOWER instead.

# Simple Searches | Specific tuning
## AND
Most BDMS operate from left to right. you can take advantage of this by putting the less likely condition on the left that way the query wont have to do the second comparison.
```sql
... WHERE column1 = 'A' AND column2 = 'B'
```

if assuming `column2 = 'B'` is less likely then the following will run faster.
```sql
... WHERE column2 = 'B' AND column1 = 'A'
```

## OR
When you're writing expressions with OR, put the most likely expression at the left. That's the exact reverse of the advice for AND, because an OR causes further tests if the first expression is false while the opposite is true for AND.

```sql
... WHERE column2 = 'B' OR column1 = 'A'
```
if assuming `column1 = 'A'` is most likely then the following will run faster.
```sql
... WHERE column1 = 'A' OR column2 = 'B'
```

### AND Plus OR
Although `A AND (B OR C)` is the same thing as `(A AND B) OR (A AND C)` but in terms of querying one will have the A comparison twice. Therefore:
```sql
... WHERE A AND (B OR C)
```
is better than 
```sql 
... WHERE (A AND B) OR (A AND C)
```

## NOT
Avoid using the NOT expression whenever you can. Futhermore Transform a NOT expression to something more readable. A simple condition can be transformed by reversing the comparison operator, for example:
```sql
... WHERE NOT (column1 > 5)
```

transforms to:
```sql
... WHERE column1 <= 5
```

## IN
between the 2 queries

```sql
... WHERE column1 = 5
    OR column1 = 6
```
and
```sql
... WHERE column1 IN (5, 6)
```
They produce the same result, but the `IN` statement is slightly more faster than the equal in some DBMS although most databases transform `IN` back to `OR` when execution.

When an IN operator has a dense series of integers, it's better to ask "what is out" rather than "what is in." Thus, this condition:
```sql
... WHERE column1 IN (1, 3, 4, 5)
```
should be transformed to:
```sql
... WHERE column1 BETWEEN 1 AND 5
      AND column1 <> 2
```

## LIKE
Use the equals operator instead of LIKE if the parameter does not contain a wildcard

```sql
... WHERE column1 LIKE 'ABC'
```

into:

```sql
... WHERE column1 = 'ABC'
```

The trap here is that LIKE 'A' and = 'A' are not precisely the same conditions. In standard SQL, a LIKE comparison takes trailing spaces into account, while an equals comparison ignores trailing spaces. Furthermore, LIKE and equals don't necessarily use the same collations by default. So don't do the transform on VARCHAR columns, and be sure to force the same collation if necessary.


## SIMILAR TO
If two expressions you're joining with OR are on columns defined as CHAR or VARCHAR, a new SQL operator might be faster than OR. The `SIMILAR TO` operator. SIMILAR is similar grep utility in Unix.

`SIMILAR` is much faster than the `OR` logic meaning
```sql
... WHERE column1 = 'A'
       OR column1 = 'B'
       OR column1 = 'K'
```
can be improved by using
```sql
... WHERE column1 SIMILAR TO '[ABK]'
```

## CASE
Suppose a search condition has more than one reference to a slow routine:

```sql
... WHERE slow_function(column1) = 3
       OR slow_function(column1) = 5
```
To avoid executing slow_function twice, transform the condition with CASE:

```sql
... WHERE 1 =
       CASE slow_function(column1)
WHEN 3 THEN 1
          WHEN 5 THEN 1
       END
```

## Summary
- When everything else is equal, DBMSs will evaluate a series of ANDed expressions from left to right (except Oracle, which evaluates from right to left). Take advantage of this behavior by putting the least likely expression first. If two expressions are equally likely, put the least complex expression first.
- Put the most likely expression first in a series of ORed expressions—unless you're using Oracle. Put the same columns together in a series of ORed expressions.
- Apply the Distributive Law to write simple search conditions with the form `A AND (B ORC)` rather than `(A AND B) OR (A AND C)`.
- Transform a NOT expression to something more readable. For a simple condition, reverse the comparison operator. For a more complex condition, apply DeMorgan's Theorem.
When you know the distribution of a set of values, you can speed things up by transforming not equals searches to greater than and less than searches.
- Transform a series of ORed expressions on the same column to IN.
- When IN has a dense series of integers, ask "what is out" rather than "what is in."
- Most DBMSs will use an index for a LIKE pattern that starts with a real character but will avoid an index for a pattern that starts with a wildcard. Don't transform LIKE conditions to comparisons with >=, <, and so on unless the LIKE pattern is a parameter, for example, LIKE ?.
- Speed up LIKE ?, where the parameter does not contain a wildcard, by substituting the equals operator for LIKE as long as trailing spaces and different collations aren't a factor.
- LIKE will always beat multiple SUBSTRINGs, so don't transform.
- Transform UNION to OR.
- Put a search condition in a CASE expression if the result is a reduction in the number of references. 
- Use CASE expressions for final filtering in the select list.
<br>

# ORDER BY
