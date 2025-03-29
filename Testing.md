
# 1. Testing docker

#### Testing Docker is up & running by running the docker's Hello world.
```bash
(ersilia) omar@DESKTOP-LRJOITF:/mnt/c/Users/a/Projects/ersilia-os$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
e6590344b1a5: Pull complete
Digest: sha256:7e1a4e2d11e2ac7a8c3f768d4166c2defeb09d2a750b010412b6ea13de1efb19
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.    

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.     
    (amd64)
 3. The Docker daemon created a new container from that image which runs the  
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:        
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

  ersilia catalog --hub
```


To demonstrate the integration of Docker with Ersilia, I can use the ersilia standard workflow of fetching models by running `fetch` followed by `serve` and `run` alternatively, I can directly hit `docker run` the repository name (ersiliaos/eos4e40), which is composed of organization `ersiliaos` name and the particular image name `eos4e40` to pull & run the image at once and it will by default pull the latest image tag(version):

```bash
$ docker run ersiliaos/eos4e40 
Unable to find image 'ersiliaos/eos4e40:latest' locally 
latest: Pulling from ersiliaos/eos4e40
69fb10dc82f9: Pull complete
14b2b354d982: Pull complete
359824f6c380: Pull complete
445db8c8475c: Pull complete
4f4fb700ef54: Pull complete
f9b97a8b35f3: Pull complete
a27fae773f91: Pull complete
c1699e87f209: Pull complete
7d738036569d: Pull complete
c00084c22505: Pull complete
Digest: sha256:9792a5d76b35cf2c608010bcbddab53e29730e1e4cf301994b7579f34275a9e2
Status: Downloaded newer image for ersiliaos/eos4e40:latest
+ [ -z eos4e40 ]
+ ersilia_model_serve --bundle_path /root/bundles/eos4e40 --port 80
2025-03-29 07:19:04,913 - ersilia_pack.utils - INFO - Serving the app from system Python
2025-03-29 07:19:04,914 - ersilia_pack.utils - INFO - /usr/local/bin/python3.9 /root/bundles/eos4e40/20241219-3d40a0a1-dc22-442a-9ce7-792e4455d134/run_uvicorn.py --host 0.0.0.0 --port 80
INFO:     Will watch for changes in these directories: ['/root/bundles/eos4e40/20241219-3d40a0a1-dc22-442a-9ce7-792e4455d134/app', '/root/bundles/eos4e40/20241219-3d40a0a1-dc22-442a-9ce7-792e4455d134/model/framework']
INFO:     Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)
INFO:     Started reloader process [8] using WatchFiles
INFO:     Started server process [10]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

- From the image pulling logs, we can see the first line image isn't found locally as expected so docker will instead pull the latest tab from the hub, the pulling starts layer by layer our `ersiliaos/eos4e40 ` image has 10 layers. `Status: Downloaded....` shows the image has been pulled **successfully**.

- `--port 80` shows that docker container is running the web service only on a port 80 server meaning it's inside the container and not mapped to my windows machine so pasting `http://localhost` into my browser won't work.
To test whether the service works or not, type `curl http://localhost` to see the respose.
Now I get :
```bash
$ curl http://localhost
curl: (7) Failed to connect to localhost port 80 after 4 ms: Connection refused
```
To fix that, I ask docker to map a port on my Windows machine by running the container this way:
```bash
 docker run -p 80:80 ersiliaos/eos4e40
```
This way the service works through the browser, and we can get a response through the terminal.

```bash
$ curl http://localhost
{"eos4e40":"chemprop-antibiotic"}
```

Another challenge we face after running Ersilia models as containers is that we want to execute model commands inside the running container!

This is caused by the follwing line in the `docker-entrypoint.sh` script that runs when the container starts:
```sh
nginx -g 'daemon off;'
```
as it keeps Nginx running in the foreground instead of detaching it as a background process.

To solve it we can either:
- Add the `-d` flag to run the container in the background.
- for an interactive shell inside the container `docker run -it --entrypoint /bin/bash ersiliaos/eos4wt0` ( I prefer that)

Note: the `-it` flag stands for interactive terminal.
Now, we can explore the container properties like the python version the model was writtin with and so many ....
```bash
root@e4e65ea2f401:~# python -V
Python 3.7.17
root@e4e65ea2f401:~# 
```

or running ```bash docker image inspect 311c``` for more detailed metadata about the container.

# 2- Testing Ersilia

#### This step verifies that the Ersilia package is installed✔️ by running a test script.  

```bash
python test_ersilia.py
{'install_status_file': '/home/omar/eos/.install.status', 'status': 'installed'}  
```  

Code in [`test_ersilia.py`](https://github.com/OmarAI2003/Ersilia-utilities/blob/main/test_installation.py) that generates the output:

```python  
import ersilia  

result = ersilia.check_install_status()  
print(result)  
```
___

To explore different Ersilia models in the hub I used `ersilia catalog --hub` to see the models there. It returned exactly 172 models in the form of a table by default:

```bash
┌───────┬────────────┬─────────────────────────────────────┐
│ Index | Identifier | Slug                                │
├───────┼────────────┼─────────────────────────────────────┤
│ 1     | eos6ost    | reinvent4-libinvent                 │
├───────┼────────────┼─────────────────────────────────────┤
│ 2     | eos3cf4    | molfeat-chemgpt                     │
├───────┼────────────┼─────────────────────────────────────┤
                    .... all the way to:

├───────┼────────────┼─────────────────────────────────────┤
│ 172   | eos1xje    | biogpt-embeddings                   │
└───────┴────────────┴─────────────────────────────────────┘
```

There wasn't enough information about models, so I added `--more` flag to fetch additional columns like input/output shape and task type. I also used the `--as-json` like this `ersilia catalog --hub --more --as-json > models_catalog.json`
to save the metadata in a [`models_catalog.json` file](https://github.com/OmarAI2003/Ersilia-utilities/blob/main/models_catalog.json) for later exploration.

Unfortunately, the catalog command doesn't show model sizes. I couldn't find a command or a way to check hub model sizes. To get around this, I used the `ersilia test <model>`, which requires installing `ersilia[test]`, and the model has to be installed locally.

So running `ersilia test eos4wt0` will start `eos4wt0` testing, and at the end, we can spot this `Model Size` table:

```bash
             Model Directory Sizes
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━┓
┃ Check                          ┃       Size ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━┩
│ Directory Size Mb              │       1.00 │
└────────────────────────────────┴────────────┘
```

@GemmaTuron Lastly, I suggest  adding an option to the `catalog` command  that provides model sizes. It would definitely benefit users who are selecting models based on their size.
_______________________________________________________

Ersilia has a different model Types:

- Pre-trained Literature Models, developed externally with pre-trained parameters.
- Re-trained Models: where Ersilia re-trains models lacking pre-trained parameters.
- In-house Developed Models, created using public datasets for infectious and neglected diseases.
- Collaboratively Developed Models: built with experimental scientists for specific research needs.
- 
I will use the antibiotic potential predictor `eos4e40` model as an example to explore a random Ersilia model.
Oversimplifiedly, `eos4e40` tests the acceptability of a small molecule to be as an antibiotic through its effect on the famous escherichia coli bacteria in other words will the molecule kill or even inhibit E.coli growth? (Model outputs a single value: probability)


To explore an Ersilia model, I try these three commands:

1) `ersilia info` (requires a served model)
- ```bash   
    $ ersilia serve eos4e40
    🚀 Serving model eos4e40: chemprop-antibiotic

    URL: http://0.0.0.0:39583
    PID: -1
    SRV: pulled_docker
    Output source: local-only

    👉 To run model:
    - run

    💁 Information:
    - info

    $ ersilia info
    🚀 Broad spectrum antibiotic activity
    Based on a simple E.coli growth inhibition assay, the authors trained a model capable of identifying antibiotic potential in compounds structurally divergent from conventional antibiotic drugs. One of the predicted active molecules, Halicin (SU3327), was experimentally validated in vitro and in vivo. Halicin is a drug under development as a treatment for diabetes.

    💁 Identifiers
    Model identifiers: eos4e40
    Slug: chemprop-antibiotic

    🤓 Code and parameters
    GitHub: https://github.com/ersilia-os/eos4e40
    AWS S3: https://ersilia-models-zipped.s3.eu-central-1.amazonaws.com/eos4e40.zip

    🐋 Docker
    Docker Hub: https://hub.docker.com/r/ersiliaos/eos4e40
    Architectures: AMD64,ARM64

    For more information, please visit https://ersilia.io/model-hub
  ```

2) `ersilia catalog --card  <model_identifier>` (requires the model to be installed already)
- ```bash
    $ ersilia catalog --card eos4e40
    {
        "card": {
            "Identifier": "eos4e40",
            "Slug": "chemprop-antibiotic",
            "Status": "Ready",
            "Title": "Broad spectrum antibiotic activity",
            "Description": "Based on a simple E.coli growth inhibition assay, the authors trained a model capable of identifying antibiotic potential in compounds structurally divergent from conventional antibiotic drugs. One of the predicted active molecules, Halicin (SU3327), was experimentally validated in vitro and in vivo. Halicin is a drug under development as a treatment for diabetes.\n\n",
            "Mode": "Pretrained",
            "Input": [
                "Compound"
            ],
            "Input Shape": "Single",
            "Task": [
                "Classification"
            ],
            "Output": [
                "Probability"
            ],
            "Output Type": [
                "Float"
            ],
            "Output Shape": "Single",
            "Interpretation": "Probability that a compound inhibits E.coli growth. The inhibition threshold was set at 80% growth inhibition in the training set.",
            "Tag": [
                "E.coli",
                "IC50",
                "Antimicrobial activity",
                "Chemical graph model"
            ],
            "Publication": "https://pubmed.ncbi.nlm.nih.gov/32084340/",
            "Source Code": "http://chemprop.csail.mit.edu/checkpoints",
            "License": "MIT",
            "Contributor": "miquelduranfrigola",
            "S3": "https://ersilia-models-zipped.s3.eu-central-1.amazonaws.com/eos4e40.zip",
            "DockerHub": "https://hub.docker.com/r/ersiliaos/eos4e40",
            "Docker Architecture": [
                "AMD64",
                "ARM64"
            ]
        },
        "model_id": "eos4e40",
        "Slug": "chemprop-antibiotic",
        "api_list": [
            "_run",
            "run"
        ],
        "service_class": "pulled_docker",
        "size": 2627.5405168533325
    }
  ```

1) `ersilia example <model_identifier>` (requires a served model)

Generates sample input examples for your model
- ```bash
   ersilia example eos4e40 --n_samples 5 --simple -f eos4e40_5_simple_examples.json
    ```

`example` Will generate 5 samples to test the model. By default, they are printed to the terminal, but I can also save them to a file via the flag `-f`. Leaving the file extension unspecified in the `-f` flag saves the result as a txt file by default. Accepted formats are .csv, .tsv, and .json, but I set it to JSON.

I then used the samples generated by the `example` command to test the model by running `$ ersilia run --input eos4e40_5_simple_examples.json`
 
but I got this **error**:

```bash
Traceback (most recent call last):
  File "/home/omar/miniconda3/bin/ersilia", line 8, in <module>
    sys.exit(cli())
             ^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/click/core.py", line 1161, in __call__
    return self.main(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/click/core.py", line 1082, in main
    rv = self.invoke(ctx)
         ^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/click/core.py", line 1697, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/click/core.py", line 1443, in invoke
    return ctx.invoke(self.callback, **ctx.params)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/click/core.py", line 788, in invoke
    return __callback(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/ersilia/cli/commands/__init__.py", line 31, in wrapper
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/ersilia/cli/commands/run.py", line 77, in run
    for result in mdl.run(
                  ^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/ersilia/core/model.py", line 365, in _api_runner_iter
    for result in api.post(input=input, output=output, batch_size=batch_size):
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/ersilia/serve/api.py", line 184, in post
    unique_input, mapping = self._unique_input(input)
                            ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/omar/miniconda3/lib/python3.12/site-packages/ersilia/serve/api.py", line 423, in _unique_input
    key = inp["key"]
          ~~~^^^^^^^
KeyError: 'key'
```
I experimented with modifying the structure of the JSON file by adding a `key` field, but this resulted in `null` in the output section. I also tested several different formats, but none of them worked except for this simple list of `str`'s structure:  

```json
[
    "CN(C)CCN1C(=O)c2cccc3cc(NC(N)=O)cc(C1=O)c23",
    "OC(COc1cccc2oc(cc(=O)c12)C(O)=O)COc1cccc2oc(cc(=O)c12)C(O)=O",
    "COC(=O)[C@H](NC(=O)c1cc(nc2ccccc12)-c1ccccc1)c1ccccc1",
    "N[C@H](Cc1cc(I)c(Oc2ccc(O)c(I)c2)c(I)c1)C(O)=O",
    "COc1cc(NC(=O)c2ccccn2)ccc1Cl"
]
```

Rerunning the model with that new file structure: `ersilia run --input eos4e40_5_simple_examples.json --as_table` worked successfully✔️ and produced this expected output:

```bash
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ key                           ┃ input                                                          ┃ outcome                 ┃
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ WCKZRLOUKYFJDY-UHFFFAOYSA-N   ┃ CN(C)CCN1C(=O)c2cccc3cc(NC(N)=O)cc(C1=O)c23                    ┃ 0.02867272967705503     ┃
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ IMZMKUWMOSJXDT-UHFFFAOYSA-N   ┃ OC(COc1cccc2oc(cc(=O)c12)C(O)=O)COc1cccc2oc(cc(=O)c12)C(O)=O   ┃ 0.04304730389849283     ┃
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ IUMQXQJZIHWLIN-HSZRJFAPSA-N   ┃ COC(=O)[C@H](NC(=O)c1cc(nc2ccccc12)-c1ccccc1)c1ccccc1          ┃ 0.01709034313680604     ┃
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ AUYYCJSJGJYCDS-GFCCVEGCSA-N   ┃ N[C@H](Cc1cc(I)c(Oc2ccc(O)c(I)c2)c(I)c1)C(O)=O                 ┃ 0.0018727317687138977   ┃
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ ARYUXFNGXHNNDM-UHFFFAOYSA-N   ┃ COc1cc(NC(=O)c2ccccn2)ccc1Cl                                   ┃ 0.007303663597122067    ┃
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
(base) omar@DESKTOP-LRJOITF:/mnt/c/Users/A/Desktop/Ersilia-utilities$ 
```
Would it make sense @GemmaTuron to either adjust the model to accept the existing output of the example command or modify the example generation process to produce a compatible format? Additionally, clarifying the expected input format in the documentation could help avoid confusion.

-------------
[`models_catalog.json` file](https://github.com/OmarAI2003/Ersilia-utilities/blob/main/models_catalog.json) Insights:

Compound: A chemical structure or molecule.

Descriptor: Computed features or fingerprints that describe molecular properties.

Probability: A prob typically used in classification tasks.

Experimental value: Prediction to mimic experimental Measurements where lab data isn't available to allow researchers to use them as proxies for actual experimental values

Score: Chemical metric scores which can indicate things like toxicity and so on...

Value: A generic numeric or categorical value to represent computed properties.

Text.

Image: Some models generate visual representations, such as structure images.

Distance.

Boolean: Returns a true/false result, probably for binary classification tasks.
_____________________
