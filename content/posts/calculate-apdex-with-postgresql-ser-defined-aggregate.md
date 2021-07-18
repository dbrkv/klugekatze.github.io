---
title: "Create User-Defined PostgreSQL Aggregate to calculate Apdex"
date: "2021-07-18"
tags:
    - database
    - postgresql
    - sql
---

I need a quick and easy Apdex score calculation, existing SQL code works, but it's hard to reuse and maintain.

In this article we will see how to create custom aggregate function for PostgreSQL to calculate Apdex score.

### What is Apdex?

Apdex is an industry standard to measuse user's satisfaction with response time of web applications and services. I'm using formula from [Wikipedia](https://en.wikipedia.org/wiki/Apdex):

```txt
(SatisfiedSount * (ToleratingCount / 2) + (FrustratedCount)) / TotalSamples
```

## General information  about PosrgreSQL aggregate functions

[Aggregate function](https://www.postgresql.org/docs/9.5/xaggr.html) is called for each row of the table, between calls she maintein an internal state that defines context for function execution. At the end of execution, aggregate function return final value.

## Write aggregate function

Let's take closer look into `CREATE AGGREGATE`

```sql
CREATE AGGREGATE apdex(float, float) (
    SFUNC = <<< function name called for each row >>>,
    STYPE = <<< type of data container to maintain state>>>,
    FINALFUNC = <<< function name to calculate final state value >>>,
    INITCOND = <<< default value for data container >>> 
    );
```

So, we need to create custom type for state value and 2 functions to process data.

```sql
CREATE TYPE apdex_state AS
(
    satisfied     float,
    tolerated     float,
    frustrated    float,
    samples_count integer
);
```

Transition function called for each row. We compare value to threshold and increment particular state property.

```sql
CREATE OR REPLACE FUNCTION apdex_transition(state apdex_state, val float, threshold float)
    RETURNS apdex_state AS
$$

BEGIN

    IF val <= threshold THEN
        RETURN (
                state.satisfied + 1,
                state.tolerated,
                state.frustrated,
                state.samples_count + 1
            )::apdex_state;
    ELSEIF val > threshold AND val <= (threshold * 4) THEN
        RETURN (
                state.satisfied,
                state.tolerated + 1,
                state.frustrated,
                state.samples_count + 1
            )::apdex_state;
    ELSEIF val > (threshold * 4) THEN
        RETURN (
                state.satisfied,
                state.tolerated,
                state.frustrated + 1,
                state.samples_count + 1
            )::apdex_state;
    END IF;

END;

$$
    LANGUAGE plpgsql IMMUTABLE;
```

Final value processing function takes  `state apdex_state` as an argument and applies formula to calculate Apdex score.

```sql
CREATE OR REPLACE FUNCTION apdex_final(state apdex_state)
    RETURNS float AS
$$
BEGIN
    IF state.samples_count = 0 THEN
        RETURN 0;
    END IF;
    RETURN (
            (state.satisfied + (state.tolerated / 2) + state.frustrated) / state.samples_count
        )::float;
END;
$$
    LANGUAGE plpgsql IMMUTABLE;
```

Create aggregate with names of functions and new type defined above

```sql
CREATE AGGREGATE apdex(float, float) (
    SFUNC = apdex_transition,
    STYPE = apdex_state,
    FINALFUNC = apdex_final,
    INITCOND = '(0, 0, 0, 0)'
    );
```

That's all.

Now, instean of complex SQL we will use `apdex(value, threshold)`, like this:

```sql
SELECT apdex(total_time, 0.7) AS apdex_score FROM requests;

    apdex_score     
--------------------
 0.8409090909090909
(1 row)
```

Works like a charm!
Awesome!

Full source code of an aggregate:

```sql
DROP TYPE apdex_state;
CREATE TYPE apdex_state AS
(
    satisfied     float,
    tolerated     float,
    frustrated    float,
    samples_count integer
);


CREATE OR REPLACE FUNCTION apdex_transition(state apdex_state, val float, threshold float)
    RETURNS apdex_state AS
$$

BEGIN

    IF val <= threshold THEN
        RETURN (
                state.satisfied + 1,
                state.tolerated,
                state.frustrated,
                state.samples_count + 1
            )::apdex_state;
    ELSEIF val > threshold AND val <= (threshold * 4) THEN
        RETURN (
                state.satisfied,
                state.tolerated + 1,
                state.frustrated,
                state.samples_count + 1
            )::apdex_state;
    ELSEIF val > (threshold * 4) THEN
        RETURN (
                state.satisfied,
                state.tolerated,
                state.frustrated + 1,
                state.samples_count + 1
            )::apdex_state;
    END IF;

END;

$$
    LANGUAGE plpgsql IMMUTABLE;


CREATE OR REPLACE FUNCTION apdex_final(state apdex_state)
    RETURNS float AS
$$
BEGIN
    IF state.samples_count = 0 THEN
        RETURN 0;
    END IF;
    RETURN (
            (state.satisfied + (state.tolerated / 2) + state.frustrated) / state.samples_count
        )::float;
END;
$$
    LANGUAGE plpgsql IMMUTABLE;

DROP AGGREGATE IF EXISTS apdex(float, float);
CREATE AGGREGATE apdex(float, float) (
    SFUNC = apdex_transition,
    STYPE = apdex_state,
    FINALFUNC = apdex_final,
    INITCOND = '(0, 0, 0, 0)'
    );

```
