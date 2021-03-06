# Concepts 

Validation of the code under test (the tested logic of procedure/function etc.) is performed by comparing the actual data against the expected data.
To achieve that, we use a combination of expectation and matcher to perform the check on the data.

Example of a unit test procedure body.
```sql
begin
  ut.expect( 'the tested value' ).to_( equal('the expected value') );
end;
```

Expectation is a set of the expected value(s), actual values(s) and the matcher(s) to run on those values.

Matcher defines the comparison operation to be performed on expected and actual values.
Pseudo-code:
```
  ut.expect( a_actual {data-type} ).to_( {matcher} );
  ut.expect( a_actual {data-type} ).not_to( {matcher} );
```

All matchers have shortcuts like:
```
  ut.expect( a_actual {data-type} ).to_{matcher};
  ut.expect( a_actual {data-type} ).not_to_{matcher};
```

# Matchers
utPLSQL provides the following matchers to perform checks on the expected and actual values.  

- `be_between`
- `be_empty`
- `be_false`
- `be_greater_than`
- `be_greater_or_equal`
- `be_less_or_equal`
- `be_less_than`
- `be_like`
- `be_not_null`
- `be_null`
- `be_true`
- `equal`
- `match`

## be_between
Validates that the actual value is between the lower and upper bound.

Example:
```sql
begin
  ut.expect( a_actual => 3 ).to_be_between( a_lower_bound => 1, a_upper_bound => 3 );
  ut.expect( 3 ).to_be_between( 1, 3 );
  --or
  ut.expect( a_actual => 3 ).to_( be_between( a_lower_bound => 1, a_upper_bound => 3 ) );
  ut.expect( 3 ).to_( be_between( 1, 3 ) );  
end;
```

## be_empty
Unary matcher that validates if the provided dataset is empty.

Usage:
```sql
procedure test_if_cursor_is_empty is
  l_cursor sys_refcursor;
begin
  open l_cursor for select * from dual where 1 = 0;
  ut.expect( l_cursor ).to_be_empty();
  --or
  ut.expect( l_cursor ).to_( be_empty() );
end;
```

When used with anydata, it is only valid for collection data types.

## be_false
Unary matcher that validates if the provided value is false.

Usage:
```sql
begin
  ut.expect( ( 1 = 0 ) ).to_be_false();
  --or 
  ut.expect( ( 1 = 0 ) ).to_( be_false() );
end;
```

## be_greater_or_equal
Checks if the actual value is greater or equal than the expected.

Usage:
```sql
begin
  ut.expect( sysdate ).to_be_greater_or_equal( sysdate - 1 );
  --or
  ut.expect( sysdate ).to_( be_greater_or_equal( sysdate - 1 ) );
end;
```

## be_greater_than
Checks if the actual value is greater than the expected.

Usage:
```sql
begin
  ut.expect( 2 ).to_be_greater_than( 1 );
  --or 
  ut.expect( 2 ).to_( be_greater_than( 1 ) );
end;
```

## be_less_or_equal
Checks if the actual value is less or equal than the expected.

Usage:
```sql
begin
  ut.expect( 3 ).to_be_less_or_equal( 3 );
  --or 
  ut.expect( 3 ).to_( be_less_or_equal( 3 ) );
end;
```

## be_less_than
Checks if the actual value is less than the expected.

Usage:
```sql
begin
  ut.expect( 3 ).to_be_less_than( 2 );
  --or 
  ut.expect( 3 ).to_( be_less_than( 2 ) );
end;
```


## be_like
Validates that the actual value is like the expected expression.

Usage:
```sql
begin
  ut.expect( 'Lorem_impsum' ).to_be_like( a_mask => '%rem#_%', a_escape_char => '#' );
  ut.expect( 'Lorem_impsum' ).to_be_like( '%rem#_%', '#' );
  --or 
  ut.expect( 'Lorem_impsum' ).to_( be_like( a_mask => '%rem#_%', a_escape_char => '#' ) );
  ut.expect( 'Lorem_impsum' ).to_( be_like( '%rem#_%', '#' ) );
end;
```

Parameters `a_mask` and `a_escape_char` represent valid parameters of the [Oracle LIKE condition](https://docs.oracle.com/database/121/SQLRF/conditions007.htm#SQLRF52142)


## be_not_null
Unary matcher that validates if the actual value is not null.

Usage:
```sql
begin 
  ut.expect( to_clob('ABC') ).to_be_not_null();
  --or 
  ut.expect( to_clob('ABC') ).to_( be_not_null() );
  --or 
  ut.expect( to_clob('ABC') ).not_to( be_null() );
end;
```

## be_null
Unary matcher that validates if the actual value is null.

Usage:
```sql
begin
  ut.expect( cast(null as varchar2(100)) ).to_be_null();
  --or 
  ut.expect( cast(null as varchar2(100)) ).to_( be_null() );
end;
```

## be_true
Unary matcher that validates if the provided value is true.
- `boolean`

Usage:
```sql
begin 
  ut.expect( ( 1 = 1 ) ).to_be_true();
  --or 
  ut.expect( ( 1 = 1 ) ).to_( be_true() );
end;
```

## equal

The equal matcher is a very restrictive matcher. It only returns true if the compared data-types are the same.
That means that comparing varchar2 to a number will fail even if the varchar2 contains the same number.
This matcher is designed to capture changes of data-type, so that if you expect your variable to be a number and it is now some other type, the test will fail and give you early indication of a potential problem.

Usage:
```sql
declare
  x varchar2(100);
  y varchar2(100);
begin
  ut.expect( 'a dog' ).to_equal( 'a dog' );
  ut.expect( a_actual => y ).to_equal( a_expected => x, a_nulls_are_equal => true );
  --or 
  ut.expect( 'a dog' ).to_( equal( 'a dog' ) );
  ut.expect( a_actual => y ).to_( equal( a_expected => x, a_nulls_are_equal => true ) );
end;
```
The `a_nulls_are_equal` parameter controls the behavior of a `null=null` comparison (**this comparison by default is true!**)

### Excluding columns and attributes from comparison 

The `equal` matcher accepts an additional parameter `a_exclude` when used to compare data of `cursor`, `object` or `table type`. 

This parameter can take three forms:
1. A `varchar2` containing comma separated names of columns/attributes to exclude
2. A `varchar2` containing **XPath** expression that lists items to be excluded
3. A `ut_varchar2_list` containing list of columns/attributes to exclude
 
The column/attribute names are **case sensitive** and cannot be quoted.
If the `a_exclude` parameter is not specified, whole cursor/object/table type is compared. 
If a column/attribute to be excluded does not exist it is simply ignored (no error).

This is useful when testing elements data that cannot be determined or set up by the tests (like `sysdate` populated by default on audit columns).

```sql
procedure test_cursors_skip_columns is
  l_expected sys_refcursor;
  l_actual sys_refcursor;
begin
  open l_expected for select 'text' ignore_me, d.* from user_tables d;
  open l_actual for select sysdate "ADate", d.* from user_tables d;
  ut.expect( l_actual ).to_equal( l_expected, a_exclude => 'IGNORE_ME,ADate' );
end;
```

```sql
create or replace type employee as object(
 first_name varchar2(50),
 last_name  varchar2(50),
 hire_date  date,
 created_at timestamp,
 created_by varchar2(30),
 modified_at timestamp,
 modified_by varchar2(50)
);

procedure test_object_skip_columns is
  l_expected employee;
  l_actual employee;
begin
  l_expected := employee('John'||rownum, 'Doe', sysdate, systimestamp, 'me', systimestamp, 'me');
  -- the actual should normally be returned by the tested code.
  l_actual   := employee('John'||rownum, 'Doe', sysdate, systimestamp, 'me', systimestamp, 'me');
  
  -- test the data excluding attributes specified by XPath
  ut.expect( anydata.convertObject(l_actual) ).to_equal( 
    anydata.convertObject(l_expected), a_exclude => '/EMPLOYEE/CREATED_AT|/EMPLOYEE/MODIFIED_AT' 
  );
end;
```

### Using cursors to compare PLSQL records on Oracle 12c

There is a great article by Tim Hall on [using the TABLE Operator with Locally Defined Types in PL/SQL](https://oracle-base.com/articles/12c/using-the-table-operator-with-locally-defined-types-in-plsql-12cr1).
If you are on Oracle 12c, you can benefit from this feature to make comparison of PLSQL records and tables super-simple in utPLSQL.
You can use the feature described in the article to convert PLSQL records and collection types to cursors. Complex cursor data can then be compared in utPLQL.  


### Comparing cursor data containing DATE fields 

**Important note**

utPLSQL uses XMLType internally to represent rows of the cursor data. This is by far the most flexible method and allows comparison of cursors containing LONG, CLOB, BLOB, user defined types and even nested cursors.
Due to the way Oracle handles DATE data type when converting from cursor data to XML, utPLSQL has no control over the DATE formatting.
The NLS_DATE_FORMAT setting from the moment the cursor was opened determines the formatting of dates used for cursor data comparison.
By default, Oracle NLS_DATE_FORMAT is timeless, so data of DATE datatype, will be compared ignoring the time component.

You should use procedures `ut.set_nls`, `ut.reset_nls` around cursors that you want to compare in your tests.
This way, the DATE data in cursors will be properly formatted for comparison using date-time format.

The example below makes use of `ut.set_nls`, `ut.reset_nls`, so that the date in `l_expected` and `l_actual` is compared using date-time formatting.  
```sql
create table events (
  description varchar2(4000),
  event_date  date
);

create or replace function get_events(a_date_from date, a_date_to date) return sys_refcursor is
  l_result sys_refcursor;
begin
  open l_result for
    select description, event_date
      from events
     where event_date between a_date_from and a_date_to;
  return l_result;
end;
/

create or replace package test_get_events is

  --%suite(get_events)

  --%beforeall
  procedure setup_events;

  --%test(returns event within date range)
  procedure get_events_for_date_range;

end;
/

create or replace package body test_get_events is

  gc_description constant varchar2(30) := 'Test event';
  gc_event_date  constant date := to_date('2016-09-08 06:51:22','yyyy-mm-dd hh24:mi:ss');
  procedure setup_events is
  begin
    insert into events (description, event_date)
    values (gc_description, gc_event_date);
  end;

  procedure get_events_for_date_range is
    l_expected          sys_refcursor;
    l_actual            sys_refcursor;
    l_expected_bad_date sys_refcursor;
    l_second   number := 1/24/60/60;
  begin
    ut.set_nls();
    open l_expected for select gc_description as description, gc_event_date as event_date from dual;
    open l_expected_bad_date for select gc_description as description, gc_event_date + l_second as event_date from dual;
    l_actual := get_events(gc_event_date-1, gc_event_date+1);
    ut.reset_nls();

    ut.expect(l_actual).to_equal(l_expected);                        
    ut.expect(l_actual).not_to_equal(l_expected_bad_date);
  end;

end;
/

begin
  ut.run();
end;
/

drop table events;
drop function get_events;
drop package test_get_events;
```

### Comparing user defined types and collections

The `anydata` data type is used to compare user defined objects and collections.
  
Example:
```sql
create type department as object(name varchar2(30));
/

create type departments as table of department;
/

create or replace package demo_dept as 
  -- %suite(demo)

  --%test(demo of object to object comparison)
  procedure test_department; 
  
  --%test(demo of collection comparison)
  procedure test_departments; 

end;
/

create or replace package body demo_dept as 
  procedure test_department is
    v_expected department;
    v_actual   department;
  begin
    v_expected := department('HR');
    v_actual   := department('IT');
    ut.expect( anydata.convertObject(v_expected) ).to_equal( anydata.convertObject(v_actual) );
  end;
  
  procedure test_department is
    v_expected department;
    v_actual   department;
  begin
    v_expected := departments(department('HR'));
    v_actual   := departments(department('IT'));
    ut.expect( anydata.convertCollection(v_expected) ).to_equal( anydata.convertCollection(v_actual) );
  end;

end;
/
```

This test will fail as `v_actual` is not equal `v_expected`. 

## match
Validates that the actual value is matching the expected regular expression.

Usage:
```sql
begin 
  ut.expect( a_actual => '123-456-ABcd' ).to_match( a_pattern => '\d{3}-\d{3}-[a-z]', a_modifiers => 'i' );
  ut.expect( 'some value' ).to_match( '^some.*' );
  --or 
  ut.expect( a_actual => '123-456-ABcd' ).to_( match( a_pattern => '\d{3}-\d{3}-[a-z]', a_modifiers => 'i' ) );
  ut.expect( 'some value' ).to_( match( '^some.*' ) );
end;
```

Parameters `a_pattern` and `a_modifiers` represent a valid regexp pattern accepted by [Oracle REGEXP_LIKE condition](https://docs.oracle.com/database/121/SQLRF/conditions007.htm#SQLRF00501)



# Supported data types

The matrix below illustrates the data types supported by different matchers.

|                               | be_between | be_empty | be_false | be_greater_than | be_greater_or_equal | be_less_or_equal | be_less_than | be_like | be_not_null | be_null | be_true | equal | match |
|:------------------------------|:----------:|:--------:|:--------:|:---------------:|:-------------------:|:----------------:|:------------:|:-------:|:-----------:|:-------:|:-------:|:-----:|:-----:|
| anydata                       |            |    X     |          |                 |                     |                  |              |         |     X       |   X     |         |   X   |       |
| blob                          |            |          |          |                 |                     |                  |              |         |     X       |   X     |         |   X   |       |
| boolean                       |            |          |    X     |                 |                     |                  |              |         |     X       |   X     |    X    |   X   |       |
| clob                          |            |          |          |                 |                     |                  |              |   X     |     X       |   X     |         |   X   |   X   |
| date                          |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |
| number                        |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |
| refcursor                     |            |    X     |          |                 |                     |                  |              |         |     X       |   X     |         |   X   |       |
| timestamp                     |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |
| timestamp with timezone       |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |
| timestamp with local timezone |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |
| varchar2                      |    X       |          |          |                 |                     |                  |              |   X     |     X       |   X     |         |   X   |   X   |
| interval year to month        |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |
| interval day to second        |    X       |          |          |       X         |         X           |      X           |     X        |         |     X       |   X     |         |   X   |       |



# Negating a matcher
Expectations provide a very convenient way to perform a check on a negated matcher.

Syntax to check for matcher evaluating to true:
```sql
begin 
  ut.expect( a_actual {data-type} ).to_{matcher};
  ut.expect( a_actual {data-type} ).to_( {matcher} );
end;
```

Syntax to check for matcher evaluating to false:
```sql
begin
  ut.expect( a_actual {data-type} ).not_to_{matcher};
  ut.expect( a_actual {data-type} ).not_to( {matcher} );
end;
```

If a matcher evaluated to NULL, then both `to_` and `not_to` will cause the expectation to report failure.

Example:
```sql
begin
  ut.expect( null ).to_( be_true() );
  ut.expect( null ).not_to( be_true() );
end;
```
Since NULL is neither *true* nor *not true*, both expectations will report failure.

