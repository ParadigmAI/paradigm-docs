# Basic Pipeline

Code: <a href="https://github.com/ParadigmAI/paradigm/tree/main/examples/basic" target="_blank">example/basic</a>

The basic pipeline is only meant to demonstrate how multiple Python scripts and notebooks can be seamlessly chained togther to form a pipeline using Paradigm. In practice, these files can perform tasks such as data handling, model trainging/evaluation and model deployment. 

In the <a href="https://github.com/ParadigmAI/paradigm/tree/main/examples/basic" target="_blank">example/basic</a> directory you will find three files that will be steps in the basic pipeline.

```
    - 📁 example/basic
        - 📄 p1.py
        - 📄 p2.ipynb
        - 📄 p3.py
        - 📄 requirements.p3
```

Here, `p1` is a python script, `p2` is an iPython notebook and `p3` is a simple FastAPI example. 

Note the `requirements.p3` file where we have the necessary dependencies for the script `p3`. These requirement files shoudl only be added if a script or a notebook needs additional dependecies. 

- First, pipeline is lauched with the below command
```
paradigm launch --steps p1 p2 p3 --region_name us-east-1
```
This commands basically takes care of containerizing the scripts and notebooks provided under `--steps` and uploading them to a specified container registry (a container registry is not used for local deployements).

- Next, we execute the pipeline with the below command

```
paradigm deploy --steps p1 p2 --dependencies "p2:p1,p3:p2|p1" --deployment p3 --deployment_port 8000 --output workflow.yaml --name pipeline1 --region_name us-east-1
```
`--steps` should speicify all steps, except any step that should be run as a service, e.g., an API endpoint. 

`--dependencies "p2:p1,p3:p2|p1"` defines the graph stucture (DAG) on how the steps should be run. In this example, we are stating that step `p2` is dependent on `p1` and step `p3` is dependent on both `p2` and `p1`. 

`--deployment p3` defines a service that needs to be run at the end of the pipeline. Hence, we don't mention is under `--steps`. 

`--deployment_port` is defined if the above service is exposed via a specific port internally. 

`--name` can be any name that you want to give this particualr pipeline

With that, we are done!

You can use Argo UI to observe all pipelines and get logs. For that, first make it accessible via your browser by running the below command. 

`kubectl -n paradigm port-forward deployment/argo-server 2746:2746`

Now in your local browser, go to `http://localhost:2746`

To access the service that is deployed in the previous set (for example an API endpoint), run the following code since we're working inside minikube. 

    - `minikube service deploy-p3 -n paradigm`

In case you want to delete the running service and deployment, use the following commands. `<deployment_step>` is the name of the file that has the deolyment code.

`kubectl delete deployment deploy-p3 -n paradigm`

`kubectl delete service deploy-p3 -n paradigm`