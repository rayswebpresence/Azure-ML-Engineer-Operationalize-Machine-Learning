# Operationalizing Machine Learning with Azure ML

> In this project, I train [Bank Marketing dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) <sup>1</sup> data using AutoML in Azure ML Studio. The best performing nidek  for is determined by AutoML. Next, it is  deployed to an HTTP for public consumption using an Azure Container Instance. Finally, we enable consumption using Swagger. 

## Overview


___
### Architectural Diagram
![Source Data](/assets/architectural_diagram.png "Steps for Project")

___
### Key Steps

 **Step 1. Authentication**
If operating on a personal machine, you'd create a Service Principal account and associate it with your specific workspace. In my case I did not, which is why it's omitted from the above diagram.
___

**Step 2. Create/Run Automated ML Experiment**

After Uploading source dataset, you're able to find and select it under `Registered Datasets`

![Source Data](/assets/1_registered_datasets.PNG "Bank Marketing Registered Dataset")
  * Created compute cluster using `Standard_DS12_v2` VM with 1 minimum node.
  * Configured **Task type and settings** for AutoML run:
    * Ensured *Explain best model* was selected
    * Made the *Training time(hours)* to 1.
    * Made sure *Max Concurrency* was set to 5.
    * Changed primary metric from `Accuracy` to `AUC_weighted` due to highly imbalanced data.<sup>2</sup>
    * Used *Monte Carlo cross validation* to validate data. 

Once run is completed (see below), you'll see this screen:
    ![Source Data](/assets/3_experiment_completed.PNG "Showing Experiment has completed")

The best performing model is shown below:
  ![Source Data](/assets/4_best_model.PNG "Best Model")

 
**Notes** early into the experiments run, run #7 with `MaxAbsScaler`, `LightGBM` model offered best performance; it wasn't until Run 40 that another combination (`StandardScalerWrapper, XGBoostClassifier`) edged out that earliy run by a mere .0027 difference in their AUC weighted.

___

 **Step 3** Deploy model
  * Looking at *Best Model* in the *Details* tab shows us what our best model is.
  * We will then deploy that model and enable "Authentication."
    * Deploying the model enables it to interact with the HTTP API service as well as allowing users to interact with the model itself sending POST requests
  *  Our model is deployed using an Azure Container Instance  (ACI)

Once the above is done, this message shows:
![Source Data](/assets/model_deployment_triggered.PNG "Model deployment triggered")
___

**Step 4** Enable Logging
*   We will use Python to enable Application Insights/logs (see `logs.py` script)
*   By enabling Application Insights we're able to retrieve logs, which help detect performance anomalies and show error conditions as they happen; it also includes analytics tools.

We enable `Application Insights` so that the `endpoints > voting-model` screen shows this:
![Source Data](/assets/app_insights_enabled.PNG "Showing Experiment has completed")

If we wait some time and then run `logs.py` again we can see the logs output:
![Source Data](/assets/logs.PNG "Showing Experiment has completed")

 ___
 
**Step 5** Swagger Documentation
It the `details` section we're able to find the Swagger URL, a link to a JSON file Azure provides are our deployed model. 
![Source Data](/assets/swagger_ui.PNG "Showing Experiment has completed")
That link is to our `swagger.json` file; which contains details about our deployment.
It is saved as a file in the same directory as two others.
First, there's `serve.py` which will run a Python server on port 8000; then, we run bash and execute `swagger.sh` to download the latest Swagger container and then run it on port 9000.

The GET method allows us to retrieve data from a web server; the POST method requests that a web server accepts data enclosed in the body our message.
 
Once the above is done, navigating to localhost:9000 will show Swagger's UI - after typing in the correct web address, "http://localhost:9000/swagger.json", this provides documentation of the HTTP API methods/responses for the model.
![Source Data](/assets/swagger-model.PNG "Swagger Model")

You can look at the expected input payload for executing the real-time machine learning service.
![Source Data](/assets/swagger_expected_post.PNG "Expected Post")

And here are the HTTP API responses
![Source Data](/assets/swagger-responses.PNG "HTTP API Responses")


___

**Step 6** Consume Model Endpoint
In the _consume_ tab of _Endpoints_, under _Basic consumption info_,  we're able to find two things we need for the next step:
* REST endpoint 
* Primary Key 
![Source Data](/assets/consume_basic_consumption.PNG "basic consumption info")

Using `endpoint.py` we're able to issue a POST request to the deployed model.

This provides JSON output from the model:
![Source Data](/assets/consume_json_output.PNG "Swagger Model")


___


**Step 7** Create, Publish, and Consume Pipeline
We now switch over to a Jupyter notebook. We make sure our `config.json` file is available in the same directory as our file. 

Then we set up our history container by setting some variables. 
![Source Data](/assets/pipelines_project.PNG "pipeline project")


After setting a number of other variables for our compute cluster, etc., we're able to begin running our pipeline.

The _Run Types_ of our experiments now show our Pipeline:
![Source Data](/assets/pipeline_experiment.PNG "show new experiment details")

When you open up the experiment, it shows the latest run and _Pipeline run overview_ as well as other details.
![Source Data](/assets/pipeline_running.PNG "pipeline running")

After completing pipeline run from Jupyter, it gives this message:
![Source Data](/assets/completing_pipeline_run.PNG "completed pipeline")

You can see below that in _Published Pipeline overview_ there's an active REST end point.
![Source Data](/assets/published_pipeline_rest_active.PNG "active REST pipeline")

The Run Details Widget shows steps:
![Source Data](/assets/run_details_widget.PNG "Run Details steps")

Further, we can see scheduled/completed run here:
![Source Data](/assets/scheduled_completed_runs.PNG "Scheduled completed runs")

___
**Step 8** Documentation
###### Screen Recording
You can find the screen recording in the folder "screen_recording"

For your convenience, I've also put it on an unlisted Youtube link, available here:
https://youtu.be/GRu8T03Hees

A second one which briefly goes into the notebook is also here (in case I wasn't in depth enough above):
https://youtu.be/WBEaxd9o1ao
## Future Work
There are things we can do for imbalanced datasets that would have to come before the AutoML steps, according to Microsoft's own docs <sup>3</sup>
1.Resampling to even the class imbalance, either by up-sampling the smaller classes or down-sampling the larger classes. These methods require expertise to process and analyze.
2. Review performance metrics for imbalanced data, particularly the F1 score. Exploring the precision (where higher = more false positives) and recall (where higher = fewer false negatives)  

So these would be some things which would contribute to the success of future work using this data.

Other ideas would be letting the training process exceed an hour.

I would be interested in trying to train a similar classification model that is more balanced and comparing results as well.

I did hard-code cross-validation based on an example online - I would also be interested in running again and letting AutoML choose.
In my experience, hard-coded happened to work slightly better, but I also tweaked a number of other things. So a systematic comparison of that step would also be interesting.

## References
<sup>1</sup> The data relates to direct marketing campaigns of a Portugese banking institution based on phone calls. The output variable is "y," which is a binary feature designating whether a person subscribed to a term deposit.  [Click here for more on the data source.](https://archive.ics.uci.edu/ml/datasets/bank+marketing) 

<sup>2</sup> The data is imbalanced, so we have to change the data's primary metric from `Accuracy`. See [my first project's README](https://github.com/rayswebpresence/nd_first_project)  for more on this issue.

<sup>3</sup> See the following from Azure ML Documentation for more on how to [Manage ML pitfalls](https://docs.microsoft.com/en-us/azure/machine-learning/concept-manage-ml-pitfalls).
