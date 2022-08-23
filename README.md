# How to train a model in Azure ML using components and a pipeline

This project shows how to train a Fashion MNIST model using a Azure ML components and a pipeline, and how to deploy it using a managed online endpoint. It uses MLflow for tracking and model representation.

## Blog post

To learn more about the code in this repo, check out the accompanying blog post: https://bea.stollnitz.com/blog/aml-pipeline/


## Azure setup

* You need to have an Azure subscription. You can get a [free subscription](https://azure.microsoft.com/en-us/free?WT.mc_id=aiml-42161-bstollnitz) to try it out.
* Create a [resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal?WT.mc_id=aiml-42161-bstollnitz).
* Create a new machine learning workspace by following the "Create the workspace" section of the [documentation](https://docs.microsoft.com/en-us/azure/machine-learning/quickstart-create-resources?WT.mc_id=aiml-42161-bstollnitz). Keep in mind that you'll be creating a "machine learning workspace" Azure resource, not a "workspace" Azure resource, which is entirely different!
* If you have access to GitHub Codespaces, click on the "Code" button in this GitHub repo, select the "Codespaces" tab, and then click on "New codespace."
* Alternatively, if you plan to use your local machine:
  * Install the Azure CLI by following the instructions in the [documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?WT.mc_id=aiml-42161-bstollnitz).
  * Install the ML extension to the Azure CLI by following the "Installation" section of the [documentation](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-configure-cli?WT.mc_id=aiml-42161-bstollnitz).
* In a terminal window, login to Azure by executing `az login --use-device-code`. 
* Set your default subscription by executing `az account set -s "<YOUR_SUBSCRIPTION_NAME_OR_ID>"`. You can verify your default subscription by executing `az account show`, or by looking at `~/.azure/azureProfile.json`.
* Set your default resource group and workspace by executing `az configure --defaults group="<YOUR_RESOURCE_GROUP>" workspace="<YOUR_WORKSPACE>"`. You can verify your defaults by executing `az configure --list-defaults` or by looking at `~/.azure/config`.
* You can now open the [Azure Machine Learning studio](https://ml.azure.com/?WT.mc_id=aiml-42161-bstollnitz), where you'll be able to see and manage all the machine learning resources we'll be creating.
* Although not essential to run the code in this post, I highly recommend installing the [Azure Machine Learning extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai).


## Project setup

If you have access to GitHub Codespaces, click on the "Code" button in this GitHub repo, select the "Codespaces" tab, and then click on "New codespace."

Alternatively, you can set up your local machine using the following steps.

Install conda environment:

```
conda env create -f environment.yml
```

Activate conda environment:

```
conda activate aml_pipeline_cli
```


## Training and inference on your development machine

* Run train.py by pressing F5.
* Run test.py the same way.

You can look at the logs generated by the code in these files with the following command:

```
mlflow ui
```


## Training and deployment in the cloud

```
cd aml_pipeline_cli
```

Create the compute cluster.

```
az ml compute create -f cloud/cluster-cpu.yml 
```

Create the dataset we'll use to train the model.

```
az ml data create -f cloud/data.yml 
```

Create the components.

```
az ml component create -f cloud/train.yml
az ml component create -f cloud/test.yml
```

Create and run the pipeline.

```
run_id=$(az ml job create -f cloud/pipeline-job.yml --query name -o tsv)
```

Go to the Azure ML Studio and wait until the Job completes.
You don't need to download the trained model, but here's how you would do it if you wanted to:

```
az ml job download --name $run_id --output-name "model_dir"
```

Create the Azure ML model from the output.

```
az ml model create --name model-pipeline-cli --version 1 --path "azureml://jobs/$run_id/outputs/model_dir" --type mlflow_model
```

Create the endpoint.

```
az ml online-endpoint create -f cloud/endpoint.yml
az ml online-deployment create -f cloud/deployment.yml --all-traffic
```

Invoke the endpoint.

```
az ml online-endpoint invoke --name endpoint-pipeline-cli --request-file test_data/images_azureml.json
```

Clean up the endpoint, to avoid getting charged.

```
az ml online-endpoint delete --name endpoint-pipeline-cli -y
```


## Related resources

* [Component YAML schema](https://docs.microsoft.com/en-us/azure/machine-learning/reference-yaml-component-command?WT.mc_id=aiml-42161-bstollnitz)
* [Pipeline YAML schema](https://docs.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-pipeline?WT.mc_id=aiml-42161-bstollnitz)
* [az ml job commands](https://docs.microsoft.com/en-us/cli/azure/ml/job?view=azure-cli-latest#az-ml-job-create?WT.mc_id=aiml-42161-bstollnitz)
* [Output formats for Azure CLI commands](https://docs.microsoft.com/en-us/cli/azure/format-output-azure-cli?WT.mc_id=aiml-42161-bstollnitz)
