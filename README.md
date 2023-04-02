# SQL TUNING

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
