# SQL TUNING
This course is to help use tune (increase efficiency) of our SQL queries. the solution provided from the methods bellow wont work in all the SQL databases out there but each and every one will work on atleast one of the main type Databases.

# Simple Searches
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