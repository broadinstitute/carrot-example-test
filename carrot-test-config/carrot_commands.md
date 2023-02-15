## 1. Create a software entry for this repository

First, we'll register this software with your CARROT server so CARROT can build Docker images from the repo.

```
carrot_cli software create --name example-test --description "An example CARROT test software" --repository_url https://github.com/broadinstitute/carrot-example-test
```

## 2. Create results so CARROT knows what our tests produce

CARROT needs to know about the results produced by our test so it can track them. We have two results.  One is a file containing example data, and the other is a file containing the results of comparing that data to our "truth" data.  We'll register both with CARROT here. 

```
carrot_cli result create --name "example data file" --description "A file containing example data to graph" --result_type file

carrot_cli result create --name "example comparison file" --description "A file containing data from comparing example data file to 2^x" --result_type file
```

## 3. Create a pipeline that you can use to group any example tests together

Creating a pipeline lets us group together all of our tests that test the same thing.

```
carrot_cli pipeline create --name "Example Pipeline" --description "Pipeline for demonstrating CARROT"
```

## 4. Create a template to hold our WDLs

Our tests will be based on a template, which defines the WDL we are testing and the WDL we are using to evaluate the outputs of our test WDL.

```
carrot_cli template create --pipeline "Example Pipeline" --name "Example Template" --description "Template for demonstrating CARROT" --test_wdl https://raw.githubusercontent.com/broadinstitute/carrot-example-test/1.0.0/carrot-test-config/test.wdl --eval_wdl https://raw.githubusercontent.com/broadinstitute/carrot-example-test/1.0.0/carrot-test-config/eval.wdl
```

## 5. Tell CARROT how to get your results by mapping them to your template outputs

CARROT needs to know which of the outputs from your WDLs map to results that you want to track, so we'll map our results to our template and specify which output matches which result.

```
carrot_cli template map_to_result "Example Template" "example data file" test_workflow.data_file
carrot_cli template map_to_result "Example Template" "example comparison file" eval_workflow.comparison_result
```

## 6. Create a report for viewing your results

CARROT provides functionality for generating Jupyter Notebook reports for runs of your tests.  We'll add our notebook (report.ipynb) to CARROT so we can generate reports by running it with data from runs of our tests.

```
carrot_cli report create --name "Example Report" --description "Report for demonstrating CARROT" --notebook report.ipynb
```

## 7. Map the template to the report so it will be generated automatically

CARROT can generate filled reports for runs automatically when the runs complete if you map the report to your template.

```
carrot_cli template map_to_report "Example Template" "Example Report"
```

## 8. Create a test that defines default values that will be used for every run

A test is built off of your template and defines the default inputs to the test and evaluation WDLs.  Typically this will include everything except for the docker image for the software you are testing.  

If you examine our default input json files, you will see that here we have defined some parameters for passing to our data generation function in the test, and in the evaluation we're using a special notation for CARROT to specify that the `data_file` input to our evaluation WDL will come from the `data_file` output of our test WDL.

```
carrot_cli test create --name "Example Test" --template "Example Template" --description "Test for demonstrating CARROT" --test_input_defaults test_input_defaults.json --eval_input_defaults eval_input_defaults.json
```

## 9. Run the test

Once our test is built, we run it, specifying the last of our inputs. In the case of this example, we're specifying the version of our software to use, so that CARROT will build a docker image for it and use it in our WDLs.

```
carrot_cli test run --test_input test_input.json --eval_input eval_input.json "Example Test"
```

The return data for this command should include a field called "run_id".  Make a note of that value so it can be used to retrieve data for that run.

## 10. Get the status of the run

Once we've started our run, we can check on its status, and view results if it has completed.  You will also receive an email notifying you of the run's completion if you have configured carrot_cli with your email address.

You can get information about your runs a few ways.  The two most common would be using:

```
carrot_cli run find_by_id [run_id]
```

Replace `[run_id]` with the value for `run_id` from the previous step.

You can also search for any runs for a test using:

```
carrot_cli test find_runs --sort "desc(created_at)" --limit 5 "Example Test"
```
This will give you the 5 most recent runs of your test.


## 11. Get the status of the report for your run

Once your run has finished, the report you configured will begin generating.  You can check on the status of the report generation using the following command.

```
carrot_cli run find_report_by_ids [run_id_or_name] "Example Report"
```

You will also receive an email notifying you of the report's completion if you have configured carrot_cli with your email address.