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