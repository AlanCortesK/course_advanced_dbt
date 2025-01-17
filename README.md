# Week 4 Project

## Task 1: Create an incremental model
For this task I did the following:
- Create a macro called `unit_testing_select_table` in order for incremental model to be easier to implement.
- I set the model `fct_events` to be incremental.

## Task 2: Add dbt-snowflake-monitoring package to monitor Snowflake costs
It seems that my costliest queries are the ones coming from Add dbt-snowflake-monitoring package haha.
But if I exlude those ones I get:

1. test.advanced_dbt.unique_mrr_with_unit_test_surrogate_key.7afb3ce33e
2. test.course_advanced_dbt.unique_fct_mrr_surrogate_key.deed3bc59b
3. test.advanced_dbt.not_null_mrr_with_unit_test_surrogate_key.5a4ec6dc02
4. test.advanced_dbt.dbt_utils_equality_mrr_with_unit_test_ref_unit_test_expected_output_mrr_.798c45bfbf
5. test.course_advanced_dbt.not_null_fct_mrr_surrogate_key.77b946eb8a

Seems like testing is costly, so it is very important to remove redundant tests.

The average daily cost for running the most expensive test is 0.0438.

## Task 3: Refactor mrr.sql to proactively avoid a modelneck

For this task I took the suggestion of creating an intermediate model based on `subscription_periods` CTE, so I created, documented and tested the model
`int_subscription_periods`.

# Week 3 Project

## Task 1: Identify a few redundant tests that can be removed
I added some new testing conventions so we can be reduce the redundant tests among the models.
For staging models I removed all tests except the one for the primary key as the others columns
are considered in source tests.

## Task 2: Write a custom generic test to replace a redundant singular test
For this task I created the `assert_valid_event_name` generict test.
Also:
1. An array type argument called `like_clause` was added so you could test several LIKE clauses using one test.
2. In dbt_project.yml I added the paths tests/generic/docs and tests/generic/schemas too macros paths.
    I did this so I could document the generic tests as macros using doc blocks and a yml file.
    (Otherwise dbt was not identyfing the yml file, this also could be done adding the test to the macros folder instead of the tests folders)
3. I applied this test to the source instead of the staging model.

## Task 3: Write a unit test to confirm MRR is calculated correctly
I didn't add a seed file for int_dates input, I think int_dates is really generic so it doesn't need a seed file for unit testing.
Also I documented these seeds files.

# Week 2 Project

## Task 1: Add SQLFluff to pre-commit hook
sqlfluff pre-commit hook was added succesfully.

## Task 2: Add more pre-commit hooks to enforce project conventions
Other pre-commit hooks were added including the optional one using `check-model-tags`

### Task 3: Generalize a custom macro
I took `rolling_average_7_periods` and created a new generlized macro called `rolling_aggregation_n_periods`.

This macro receives the following parameters:
1. column_name
2. partition_by
3. number_of_periods
4. aggregation_type
5. order_by

Also, the name of the columns is generated according to the parameters passed.
I documented the macro using a md file and a yml file with doc blocks.

This macro was used in a new model I created called `rpt_subscription_plans_metrics_by_month`

### Task 4: Write a custom macro to improve another part of the codebase
A transaformation that is used quite ofthen in rpt_mrr model is truncating to month, so I created the macro `trunc_to_month`.
I also included the md file and the yml file to document this macro.

This macro can accept an `alias` parameter to add an alias, or it can be left null to not include this part in the compiled code (usefull if you are using the macro in a case when or inside another function)


# Week 1 Project

## Task 2: Add docs blocks to populate missing documentation and maintain consistency
I put all the project's documentation on doc blocks.
Usually we create doc blocks for sources and then one md file for every model. In this file we write the model description
and new columns that were created within this model or columns that were changed and now the definition is different.

For example in the source `bingeflix.subscription_plans` there is a column named `pricing`, however in
`stg_bingeflix__subscription_plans` this columns is transformed using the macro `cents_to_dollars` meaning that
in the source, pricing column is in cents and in stg model (and downstream models) `pricing` is in dollars.
Therefore, in my opinion, there should be 2 doc blocks, one for the source and one for the stg model, and it is the last one
that is going to be use downstream.

## Task 3: Task 3: Install dbt_project_evaluator package to enforce best practices
After adding all documentation I only got 4 warnings:
- Naming conventions for `mrr.sql` model
- Missing primary key tests in `fct_events.sql` model
- Model fanout in `stg_bingeflix__events`
- Valid test coverage (This is related to the second one)

1. For the naming convention warning I renamed the model `mrr` to `rpt_mrr`.
    - After the renaming I got the warning "fct_model_directories", so to practice exceptions I added rpt_mrr to the exceptions seed to allow rpt models in the same directory as fct models.
2. For the tests warnings I just added the primary key test to `fct_events`.
3. For the fanout problem I realized that `fct_events` contains the same data as `stg_bingeflix__events`, so I connected `stg_bingeflix__events` downstream models to `fct_events`.

## Task 4: [OPTIONAL] Install SQLFluff and run it to fix violations
For this task I set all keywords to upper case, and a line lenght of 100, then I used `sqlfluff fix` to lint the code.


# Welcome to the Bingeflix Data Team

### Coding Conventions
#### General
- Use UPPER case for all keywords
- Use trailing commas in SELECT statements
- Use Snowflake dialect
- Use consistent style in GROUP BY and ORDER BY (either names or numbers, not both)


### Testing Conventions
#### Sources
- The primary key source column must have not_null and unique generic tests.
- All boolean columns must have an accepted_values schema test. The accepted values are true and false.
- Columns that contain category values must have an accepted_values schema test.
- Columns that should never be null must have a not_null schema test.
- Columns that should be unique must have a unique schema test.

#### Models
The following rules must be applied to all models:
- The primary key column must have not_null and unique schema tests.
- Columns that should be unique must have a unique schema test.

The following rules can be ommited if:
1. It is a staging model and the source is tested.
2. The model contain simple transformations that don't change the definition of the column
    and it is a model with only 1 dependency.
3. The column is not transformed and came from the principal table in a left or right join.
    Note: These tests must be applied for full outer joins.
4. The model contains group bys, aggregations, and more complex transformations.
Otherwise these tests are contemplated in upstream models:
- All boolean columns must have an accepted_values schema test. The accepted values are true and false.
- Columns that contain category values must have an accepted_values schema test.
- Columns that should never be null must have a not_null schema test.
- Where possible, use schema tests from the dbt_utils or dbt_expectations packages to perform extra verification.
