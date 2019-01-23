# FloydHub Tutorial (GEC Academy)

This is a brief tutorial on Floydhub intended for students in *Deep Learning Advanced Course* with GEC Academy. All students are assumed to have basic knowledge of Unix systems and encouraged to read this tutorial before using FloydHub's service. For a complete and official doc, please refer to FloydHub [docs page](https://docs.floydhub.com/). If there still are questions pertaining to the usage of FloydHub, please consult the TAs or contact FloydHub support yourself.

* [Setup](#setup)
* [Purchase GPU/CPU time and storage space](#purchase-gpucpu-time-and-storage-space)
* [Manage datasets](#manage-datasets)
* [Create projects](#create-projects)
* [Run jobs](#run-jobs)
  + [Instance type](#instance-type)
  + [Python environment and dependencies](#python-environment-and-dependencies)
  + [Datasets](#datasets)
  + [Messages](#messages)
  + [Run-commands](#run-commands)
* [Monitor jobs](#monitor-jobs)
  + [The Overview tab](#the-overview-tab)
  + [The Files tab & Input Data tab](#the-files-tab--input-data-tab)
  + [The Output tab](#the-output-tab)
  + [Tensorboard [Optional]](#tensorboard-optional)

## Setup

1. Go to FloydHub's [homepage](https://www.floydhub.com/run?template=https://github.com/floydhub/sentiment-analysis-template) and create your own account.

2. Install FloydHub command line on your local machine: open terminal and run `pip install -U floydhub-cli`. Note that if you change your python environment with anaconda, you might need to run the above install command again.

3. Login on your local machine: run  `floyd login -u <YOUR_USER_NAME>` on your terminal, where `<YOUR_USER_NAME>` should be replaced with your username when creating your account. When prompted to input password, just type in your password and hit enter. The output should look like this

   ```
   > floyd login -u <YOUR_USER_NAME>
   Login with your FloydHub username and password to run jobs.
   Password:
   Login Successful as <YOUR_USER_NAME>
   ```


## Purchase GPU/CPU time and storage space

1. After login, you can upgrade to **Data Scientist** plan [here](https://www.floydhub.com/settings/billing) if you are interested in better concurrency or longer job time.
2. Before you can run a job on FloydHub, go to this [link](https://www.floydhub.com/settings/powerups) and check your remaining GPU/CPU instance time. You need to purchase more instance time if you don't have enough of them.
3. Also, make sure you have enough storage space to hold all your uploaded datasets and job outputs. If the initial quota is not enough, purchase a subscription which can be disabled when no long needed (e.g. after the course).

## Manage datasets

To train a model, a dataset must be fed to it. You can just use the datasets made public by other users, or create your own datasets.

1. To create your own dataset, first go to [this link](https://www.floydhub.com/datasets/create) and choose your dataset name. Note that some special characters are not supported and it is preferable to use only hyphen("-"), underscore("_") and alphanumeric characters.

2. Open terminal and `cd` into the dataset folder on your local machine.

3. Run the command `floyd data init <YOUR_DATASET_NAME>` where `<YOUR_DATASET_NAME>` should be replaced by your own dataset's name.

4. Run `floyd data upload` to upload your dataset. This can take a while depending on your internet connection.

   You should see the following output

   ```
   > floyd data init <YOUR_DATASET_NAME>
   Data source "<YOUR_DATASET_NAME>" initialized in current directory
   
       You can now upload your data to Floyd by:
           floyd data upload
   
   > floyd data upload
   Get number of files to compress... (this could take a few seconds)
   Compressing 66 files
   Compressing data...
   [================================] 66/66 - 00:00:00
   Making create request to server...
   Initializing upload...
   Uploading compressed data. Total upload size: 42.7KiB
   [================================] 43740/43740 - 00:00:020
   Removing compressed data...
   Upload finished.
   Waiting for server to unpack data.
   You can exit at any time and come back to check the status with:
   	floyd data upload -r
   Waiting for unpack....
   
   NAME
   ------------------------
   <YOUR_USER_NAME>/datasets/<YOUR_DATASET_NAME>/1
   ```


To make your dataset accessible to a job, you will need to **mount** the dataset, which will be discussed later.

## Create projects

1. Before running a **job**, you need to create a **project** on FloydHub's [website](https://www.floydhub.com/projects/create) and pick a project name.

2. Write your codes locally, and save them under a local folder.

3. `cd` into the folder and run `floyd init <YOUR_PROJECT_NAME>`, where `<YOUR_PROJECT_NAME>` should be replaced with your own project's name.

4. Make sure that your code folder is not too large. A folder containing codes should be typically less then a few MB's. Every time you run a job, all files in the folder are synced to FloydHub's server. If you do not want upload some of the content under the folder, you need to create a file named  `.floydignore`, and put in it everything you don't want to upload. For example, a `.floydignore` file can look like this: 

   ```
   image_dataset/
   .git
   *.pyc
   *.jpg
   ```

   This file named `.floydignore` tells the floydhub client to ignore anything in the `image_dataset/` or `.git`sub-folder and any file than ends with .`pyc` or `.jpg`; i.e., these files will not be synced when a job is run.

## Run jobs

Finally, we can run our jobs with the `floyd run ...` command, where  `...` stands for additional specifications. A typical "floyd run" command looks like this.

```bash
floyd run --gpu --env pytorch-1.0 --data wish1104/datasets/emnist/1:data --message 'pytorch alexnet on 62 classes, batch_size = 512' 'python pytorch_models/train.py --folder /data --param_summarize_period 20'
```

This command is broken down as follows.

### Instance type

Do you want to run on a CPU instance or a GPU instance? You can specify the instance type with one of the following `--cpu`, `--gpu`, `--cpu2`, `--gpu2`. If none of them is supplied, the job will be run on a `cpu` machine by default.

Make sure you have enough instance time as jobs will be shutdown once you run out of time. One way to prevent this is to enable **auto refresh** of your instance time.

### Python environment and dependencies

What packages are imported in your program? Is it tensorflow, pytorch, mxnet or something else? You can specify this by using `--env XXX`, where `XXX` is your environment's name. When there is no such specification, `--env tensorflow` will be used as default.

For example, use `--env tensorflow` for a tensorflow environment, or `--env pytorch` for a pytorch environment. You can also specify the version of the environment with something like `--env pytorch-0.4` or `--env pytorch-1.0`. 

Many packages are installed by default, including`h5py`, `iPython`, `Jupyter`, `matplotlib`, `numpy`, `OpenCV`, `Pandas`, `Pillow`, `scikit-learn`, `scipy`, and `sklearn`. If you need to import some other libraries, you need to create a file named `floyd_requirements.txt` in the project folder. In each line of this file, write the name of an additional package. (Note that packages listed here must be available on PyPI.) 

For example the following `floyd_requirements.txt` file will install (via `pip`) `tensorboardX` (version 1.6) and default `nltk`, `gensim`, `mlxtend`.

```
tensorboardX==1.6
nltk
gensim
mlxtend
```

### Datasets

Datasets must be **mounted** for them to be accessible to jobs. The argument to mount a dataset is `--data <URL_TO_DATASET>:<MOUNT_POINT>`. Up to 5 datasets can be mounted to each job. To mount multiple datasets, just use `--data <URL_TO_DATASET>:<MOUNT_POINT>` multiple times. 

You can mount datasets either owned by you or made public by other users.

1. `<URL_TO_DATASET>` is a URL to your datset, exluding the prefix "`https://www.floydhub.com/`". For example, if your dataset URL is `https://www.floydhub.com/wish1104/datasets/emnist`, then `<URL_TO_DATASET>` should be `wish1104/datasets/emnist`.

2. `<MOUNT_POINT>` is the location to **mount** your dataset. In plain words, you can run your job as if there is a folder named `<MOUNT_POINT>` on your remote instance. 

   Note that `<MOUNT_POINT>` must be a simple folder name (not a nested subfolder). For example, you can name your `<MOUNT_POINT>` as `datafolder` (or `/datafolder`), but not `datafolder/subfolder`. 

   Regardless of whether you name it `datafolder` or `/datafolder`, it will be mounted under the root directory `/` of linux systems. So, even if you name `<MOUNT_POINT>` as `datafolder`, it is actually an absolute path `/datafolder` instead of a relative path.

Here is an example: `--data wish1104/datasets/emnist/1:data`. This code segment means that the dataset available at `https://www.floydhub.com/wish1104/datasets/emnist/1` will be mounted at  a folder called `/data` under the linux root.

 You can also mount the outputs of a previous job like this `--data wish1104/projects/character-recognition/57/output:previous-output`. In this example, mounted at the folder called `/previous-output` is the output of job `57` of a project called `character-recognition` of a FloydHub user named `wish1104`.

### Messages

A meaningful description can make it easy to understand what each job is about. To supply a description, you need to use the `--message '<YOUR_TEXT_MESSAGE>'`, where `<YOUR_TEXT_MESSAGE>` is a description message for the job. Note that the message must be surrounded by quotes. For example, `--message 'hypertuning for 50 epochs'` will add to the job a description "hypertuning for 50 epochs". 

The description, whether supplied or not, can  always be edited on your job's webpage.

### Run-commands

At the end of `floyd run ...` command is a string, surrounded by quotes, indicating what (linux) commands to be run. For example, if you run `floyd run 'pwd'`, a job will be launched and the command `pwd` (print working directory) will be run. 

More often, you want to run a python script with additional arguments. In this scenario, usage of the `argparse` library is recommended. Suppose you have the following file named `test.py` that reads:

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--x", type=int, default=0, help="value of x")
parser.add_argument("--y", type=int, default=0, help="value of y")
parser.add_argument("--z", type=int, default=0, help="value of z")
opt = parser.parse_args()

print("value of x is {}".format(opt.x))
print("value of y is {}".format(opt.y))
print("value of z is {}".format(opt.z))
```

Then, running the `floyd run --cpu --message 'testing the code' 'python test.py --x 111 --y 222'` will yield the following output:

```
value of x is 111
value of y is 222
value of z is 0
```

However, the output of the job is not directly rendered to your terminal. To read your output logs and monitor you jobs, refer to the next part of this tutorial.

## Monitor jobs

After running your job, you should see something like this on your terminal.

```
Creating project run. Total upload size: 117.7KiB
Syncing code ...
[================================] 126960/126960 - 00:00:08

JOB NAME
------------------------------------------
wish1104/projects/character-recognition/58

URL to job: https://www.floydhub.com/wish1104/projects/character-recognition/58

To view logs enter:
   floyd logs wish1104/projects/character-recognition/58
```

Open the "`URL to job`" in your browser, you will go to the job's webpage.

Here, you will see several tabs: `Overview`, `Files`,`Input Data`, `Settings`, and possibly `Output`.

### The Overview tab

![a](resources/tabs.jpg)

In the `Overview` tab, you will see a label at the top right corner of the webpage (just under your profile image), which says `Queued`, `Success`, `Running`, ` Shutdown` or `Failed`, indicating the current status of the job. 

You can also see the **system metrics** (collected every minute) about the utilization of CPU, GPU and SSD.

![a](resources/system-metrics.jpg)

Meanwhile, the **training metrics** are automatically parsed from the `stdout` of your job. 

![a](resources/training-metrics.jpg)

For a metric point to be parsed, print it in json format like this:

```python
print('{"metric": "<metric_name>", "value": <int_or_float>, "step": <int>}'). 
```

**IMPORTANT: It is imperative to print your metrics for every few epochs so that we can assess your model performance. Some metrics to be included are loss/cost, accuracy, precision, recall, F-score (if pertinent) for both training and validation phase** (Detailed instructions [here](https://docs.floydhub.com/guides/jobs/metrics/).)

Also, the `stdout` (e.g. everything you `print()` in python) will be displayed in the `logs` section, as you can see a black text field that resembles a terminal.

![a](resources/logs.jpg)

### The Files tab & Input Data tab

In the Files tab is a copy of your local project folder at the time a job is run. Not surprisingly, files/folders whose names are present in the `.floydignore` file are not inlcuded.

![a](resources/files-tab.jpg)

In the Input Data tab is a description of all datasets mounted for the job.

![a](resources/input-data-tab.jpg)

### The Output tab

This tab may not appear on the webpage if you have no output files, or output files whose total size is smaller than 20 KiB. If the tab does not appear, you can still find it by appending the string `output` to you job's URL. For example, even though the job `www.floydhub.com/wish1104/projects/character-recognition/58/` has no "Output" tab on its web page, you can go to `www.floydhub.com/wish1104/projects/character-recognition/58/output` to see its output.

![a](resources/output-tab.jpg)

__For each FloydHub instance, only outputs in the /output folder is collected. Therefore, you must always save output files to the /output folder. Otherwise, output files will not be saved or a PermissionError will occur.__

### Tensorboard [Optional]

While a job is still running, a link to a tensorboard server should appear on the job's webpage. If you know how to use tensorboard (or `tensorboardX` for PyTorch) you can instantiate `SummaryWriter`s to save summaries in the `/output` folder. 

When the job terminates (with state failed/success/shutdown), the tensorboard server is also shutdown. If you want to analyze your metrics, download the summary files in the Output tab and start a tensorboard server locally. Alternatively, start a new job (on a CPU instance for cost-efficiency), mount the output of the orignal job, `cp` all summary files to `/output` folder of the new job, and hang the job indefinitely. You can manually shutdown the job once you are finished analyzing the metrics.