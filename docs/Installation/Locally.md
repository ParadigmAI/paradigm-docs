# Try Locally 💻

You need a Kubernetes cluster and `kubectl` set up to be able to access that cluster. For this to run locally, we recommend using `minikube`.
- Please refer to the [minikube documentation](https://minikube.sigs.k8s.io/docs/)

### Set up Paradigm

- (Recommended) Create a new Python environment with your preferred environment manager
- Clone this repo 
    - `git clone https://github.com/ParadigmAI/paradigm.git`
- Go into the directory 
    - `cd paradigm`
- Make the installation script executable 
    - `chmod +x install.sh`
- Run the intallation script 
    - `./install.sh`
- Validate if paradigm was properly installed
    - `paradigm --help`

### Now let's move into your ML project folder

Your folder can contain one or more scripts or Python notebooks that you want to execute as steps in an ML pipeline.

- First, let's configure your current terminal session to use the Docker daemon inside the Minikube environment instead of the default Docker daemon on your host machine. This eliminated the need for an image registry when working locally.
    - `eval $(minikube docker-env)`

**From here we follow a basic example project just to make it easier to exaplin the commands. Please change the necessary parameters according to your project**
- The preferred directory structure should be as follows. In the below example, `p1, p2 and p3` represent the names of the python scripts or notebooks you have. (Refer the [examples/basic](./examples/basic))
    - IMPORTANT - Note the `requirements.<file name>` files. You have to create a txt with that specific naming **only for the scripts or notebooks that have additional dependencies**. It becomes the `requirements.txt` for that step. We promise this is the only file addition before taking your ML code to prodution.
    - Example:
```
    - 📁 project_root
        - 📄 p1.py
        - 📄 p2.ipynb
        - 📄 p3.py
        - 📄 requirements.p1
        - 📄 requirements.p3
```

- Now we are ready to let Paradigm get things ready before deploying to Kubernetes. Include the scripts/notebook you want as steps in the below command. This command basically containerizes your code.
```
paradigm launch --steps p1 p2 p3
```
- As the final step, deploy the pipeline with the below command.

```
paradigm deploy --steps p1 p2 --dependencies "p2:p1,p3:p2|p1" --deployment p3 --deployment_port 8000 --output workflow.yaml --name pipeline1
```
- In the above command: 
    - `--steps` should speicify all steps, except any step that should be run as a service, e.g., an API endpoint. 
    - `--dependencies "p2:p1,p3:p2|p1"` defines the graph stucture (DAG) on how the steps should be run. In this example, we are stating that step `p2` is dependent on `p1` and step `p3` is dependent on both `p2` and `p1`. 
    - `--deployment p3` defines a service that needs to be run at the end of the pipeline. Hence, we don't mention is under `--steps`. 
    - `--deployment_port` is defined if the above service is exposed via a specific port internally. 
    - `--name` can be any name that you want to give this particualr pipeline

-  (OPTIONAL) You can use Argo UI to observe all pipelines and get logs. For that, first make it accessible via your browser by running the below command. 
    - `kubectl -n paradigm port-forward deployment/argo-server 2746:2746`
    - Now I your local browser, go to `http://localhost:2746`

- (OPTIONAL) To access the service that is deployed in the previous set (for example an API endpoint), run the following code since we're working inside minikube. 

    - `minikube service deploy-p3 -n paradigm`

- (OPTIONAL) In case you want to delete the running service and deployment, use the following commands. `<deployment_step>` is the name of the file that has the deolyment code.
    - `kubectl delete deployment deploy-<deployment_step> -n paradigm`
    - `kubectl delete service deploy-<deployment_step> -n paradigm`