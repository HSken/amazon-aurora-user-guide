# Specifying DB parameters<a name="USER_ParamValuesRef"></a>

DB parameter types include the following:
+ Integer
+ Boolean
+ String
+ Long
+ Double
+ Timestamp
+ Object of other defined data types
+ Array of values of type integer, Boolean, string, long, double, timestamp, or object

You can also specify integer and Boolean parameters using expressions, formulas, and functions\. 

**Contents**
+ [DB parameter formulas](#USER_ParamFormulas)
  + [DB parameter formula variables](#USER_FormulaVariables)
  + [DB parameter formula operators](#USER_FormulaOperators)
+ [DB parameter functions](#USER_ParamFunctions)
+ [DB parameter log expressions](#USER_ParamLogExpressions)
+ [DB parameter value examples](#USER_ParamValueExamples)

## DB parameter formulas<a name="USER_ParamFormulas"></a>

A DB parameter formula is an expression that resolves to an integer value or a Boolean value\. You enclose the expression in braces: \{\}\. You can use a formula for either a DB parameter value or as an argument to a DB parameter function\.

**Syntax**  

```
{FormulaVariable}
{FormulaVariable*Integer}
{FormulaVariable*Integer/Integer}
{FormulaVariable/Integer}
```

### DB parameter formula variables<a name="USER_FormulaVariables"></a>

Each formula variable returns an integer or a Boolean value\. The names of the variables are case\-sensitive\.

*AllocatedStorage*  
Returns an integer representing the size, in bytes, of the data volume\.

*DBInstanceClassMemory*  
 Returns an integer for the number of bytes of memory available to the database process\. This number is internally calculated by taking the total amount of memory for the DB instance class and subtracting memory reserved for the operating system and the RDS processes that manage the instance\. Therefore, the number is always somewhat lower than the memory figures shown in the instance class tables in [Aurora DB instance classes](Concepts.DBInstanceClass.md)\. The exact value depends on a combination of instance class, DB engine, and whether it applies to an RDS instance or an instance that's part of an Aurora cluster\. 

*EndPointPort*  
Returns an integer representing the port used when connecting to the DB instance\.

### DB parameter formula operators<a name="USER_FormulaOperators"></a>

DB parameter formulas support two operators: division and multiplication\.

*Division operator: /*  
Divides the dividend by the divisor, returning an integer quotient\. Decimals in the quotient are truncated, not rounded\.  
Syntax  

```
dividend / divisor
```
The dividend and divisor arguments must be integer expressions\.

*Multiplication operator: \**  
Multiplies the expressions, returning the product of the expressions\. Decimals in the expressions are truncated, not rounded\.  
Syntax  

```
expression * expression
```
Both expressions must be integers\.

## DB parameter functions<a name="USER_ParamFunctions"></a>

You specify the arguments of DB parameter functions as either integers or formulas\. Each function must have at least one argument\. Specify multiple arguments as a comma\-separated list\. The list can't have any empty members, such as *argument1*,,*argument3*\. Function names are case\-insensitive\.

*IF*  
Returns an argument\.  
Syntax  

```
IF(argument1, argument2, argument3)
```
Returns the second argument if the first argument evaluates to true\. Returns the third argument otherwise\.

*GREATEST*  
Returns the largest value from a list of integers or parameter formulas\.  
Syntax  

```
GREATEST(argument1, argument2,...argumentn)
```
Returns an integer\.

*LEAST*  
Returns the smallest value from a list of integers or parameter formulas\.  
Syntax  

```
LEAST(argument1, argument2,...argumentn)
```
Returns an integer\.

*SUM*  
Adds the values of the specified integers or parameter formulas\.  
Syntax  

```
SUM(argument1, argument2,...argumentn)
```
Returns an integer\.

## DB parameter log expressions<a name="USER_ParamLogExpressions"></a>

You can set an integer DB parameter value to a log expression\. You enclose the expression in braces: \{\}\. For example:

```
{log(DBInstanceClassMemory/8187281418)*1000}
```

The `log` function represents log base 2\. This example also uses the `DBInstanceClassMemory` formula variable\. See [DB parameter formula variables](#USER_FormulaVariables)\. 

## DB parameter value examples<a name="USER_ParamValueExamples"></a>

These examples show using formulas, functions, and expressions for the values of DB parameters\.

**Note**  
DB Parameter functions are currently supported only in the console and aren't supported in the AWS CLI\.

**Warning**  
Improperly setting parameters in a DB parameter group can have unintended adverse effects\. These might include degraded performance and system instability\. Use caution when modifying database parameters and back up your data before modifying your DB parameter group\. Try out parameter group changes on a test DB instance, created using point\-in\-time\-restores, before applying those parameter group changes to your production DB instances\. 

**Example using the DB parameter function LEAST**  
You can specify the `LEAST` function in an Aurora MySQL `table_definition_cache` parameter value\. Use it to set the number of table definitions that can be stored in the definition cache to the lesser of `DBInstanceClassMemory`/393040 or 20,000\.  

```
LEAST({DBInstanceClassMemory/393040}, 20000)
```