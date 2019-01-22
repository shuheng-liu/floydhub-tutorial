# FloydHub Tutorial (GEC Academy)

This is a breif FloydHub Tutorial intended for students in *Deep Learning Advanced Course* with GEC Academy. All students are encouraged to read this tutorial before using FloydHub's service. For a complete and official doc, please refer to FloydHub [docs homepage](https://docs.floydhub.com/). If you still have any questions pertaining to the usage of FloydHub, please consult the TAs or contact FloydHub support yourself.

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
* [Monitor your jobs](#monitor-your-jobs)
  + [The Overview tab](#the-overview-tab)
  + [The Files tab & the Input Data tab](#the-files-tab--the-input-data-tab)
  + [The Output tab](#the-output-tab)
  + [Tensorboard [Optional]](#tensorboard-optional)

## Setup

1. Go to FloydHub's [Homepage](https://www.floydhub.com/run?template=https://github.com/floydhub/sentiment-analysis-template) and create your own account.
2. Install FloydHub command line on your local machine: Open terminal and run `pip install -U floydhub-cli`. Note that if you change your environment with anaconda, you might need to run the above command again.
3. Login on your local machine: Run  `floyd login -u <YOUR_USER_NAME>` on your terminal, where `<YOUR_USER_NAME>` should be replaced with your username when creating your account. When prompted to input password, just type in your password.

## Purchase GPU/CPU time and storage space

1. After login, you can Upgrade to **Data Scientist** plan [here](https://www.floydhub.com/settings/billing) if you are interested in better concurrency longer job time.
2. Before you can run a job on FloydHub, go to this [link](https://www.floydhub.com/settings/powerups) to check your remaining GPU/CPU instance time. you need to purchase some instance time if you don't have enough of them.
3. Also, make sure you have enough storage space to hold all your uploaded datasets and job outputs. If the initial quota is not enough, purchase a subscription and disable it when you no long need it (e.g. after the course).

## Manage datasets

To train a model, a dataset must be fed to it. As a FloydHub user, you can just use the datasets made public by other users, or create your own datasets.

1. To create your own dataset, first go to [this link](https://www.floydhub.com/datasets/create) and type in a dataset name. Note that some special characters are not supported and it is preferable to use only hyphen("-"), underscore("_") and alphanumeric characters.
2. Open terminal and `cd` into the dataset folder on your local machine. (I suppose you know how to do that)
3. Run the command `floyd data init <YOUR_DATASET_NAME>` where `<YOUR_DATASET_NAME>` should be replaced by your own dataset's name.
4. Run `floyd data upload` to upload your dataset. This can take a while depending on your internet connection.

To make your dataset accessible to a job, you will need to **mount** the dataset, which will be discussed later.

## Create projects

1. Before running a **job**, you need to create a **project** on FloydHub's [website](https://www.floydhub.com/projects/create).

2. Write your codes locally, and save them under a local folder.

3. `cd` into the folder and run `floyd init <YOUR_PROJECT_NAME>`, where `<YOUR_PROJECT_NAME>` should be replaced with your own project's name.

4. Make sure that your coder folder is not too large. A folder containing codes should be typically less then a few MB's. Every time you run a job, all codes are synced to FloydHub's server. If you do not want upload some of the content under the folder, you need to create a file named  `.floydignore`, and put in it everything you don't want to upload. For example, a `.floydignore` file can look like this: 

   ```
   images/
   .git
   *.pyc
   *.jpg
   ```

   This file named `.floydignore` tells the floydhub client to ignore anything in the `images/` or `.git`sub-folder  and any file than ends with .`pyc` or `.jpg`. These files will not be synced when a job is run.

## Run jobs

Finally, we can run our jobs with the `floyd run ...` command, where  `...` stands for additional specifications. A typical "floyd run" command looks like this.

```bash
floyd run --gpu --env pytorch-1.0 --data wish1104/datasets/emnist/1:data --data wish1104/datasets/pytorch-pretrained-models/1:pretrained --message 'pytorch alexnet on 62 classes, batch_size = 512' 'python pytorch_models/train.py --folder /data --batch 512 --max_steps 500 --report_period 10 --param_summarize_period 20 --pretrained /pretrained/alexnet-owt-4df8aa71.pth --cuda --checkpoint'
```

This command is broken down as follows.

### Instance type

Are you running on a CPU instance or a GPU instance? You can specify the instance type with one of the following `--cpu`, `--gpu`, `--cpu2`, `--gpu2`. If none of them is supplied, the job will be run on a `cpu` machine by default.

Make sure you have enough instance time as jobs will be shutdown once you run out of your instance time. One way to prevent this is to enable **auto refresh** of your instance time.

### Python environment and dependencies

What libraries are imported in your program? Is it tensorflow, pytorch, mxnet or something else? You can specify this by using `--env XXX`, where `XXX` is your environment's name. When there is no such specification, `--env tensorflow` will be used as default.

For example, use `--env tensorflow` for a tensorflow environment, or `--env pytorch` for a pytorch environment. You can also specify the version of the environment with something like `--env pytorch-0.4` or `--env pytorch-1.0` for pytorch v0.4 or v1.0. 

Many packages are installed by default, including `scikit-learn`, `numpy` and other common libraries used in data science. If you need to import some other libraries, you need to create a file named `floyd_requirements.txt` in the project folder. In each line of this folder, write the name of a package. Note that packages listed here must be available on PyPI. For example the following `floyd_requirements.txt` file will install (via `pip`) `tensorboardX` (version 1.6) and default `nltk`, `gensim`, `mlxtend`.

```
tensorboardX==1.6
nltk
gensim
mlxtend
```

### Datasets

Datasets must be **mounted** for them to be accessible to jobs. The argument to mount a dataset lookslike this `--data <URL_TO_DATASET>:<MOUNT_POINT>`. Up to 5 datasets can be mounted to a job. To mount multiple datasets, just use `--data <URL_TO_DATASET>:<MOUNT_POINT>` multiple times. You can mount datasets both owned by you, or made public by other users if you have the URL.

1. `<URL_TO_DATASET>` is a URL to your datset, exluding the "`https://www.floydhub.com/`" part. For example if your dataset URL is `https://www.floydhub.com/wish1104/datasets/emnist`, then `URL_TO_DATASET` should be `wish1104/datasets/emnist`.
2. `<MOUNT_POINT>` is the location to **mount** your dataset. In plain words, you can run your job as if there is a folder named `<MOUNT_POINT>` on your remote instance. Note that `<MOUNT_POINT>` must be a simple folder name (not a nested subfolder). For example, you can name your `<MOUNT_POINT>` as `datafolder` (or `/datafolder`), but not `datafolder/subfolder`. Regardless of whether you name it `datafolder` or `/datafolder`, it will be mounted under the root directory `/` of linux systems. So, even if you name `<MOUNT_POINT>` as `datafolder`, it is actually an absolute path `/datafolder`, instead of a relative path.

Here is an example: `--data wish1104/datasets/emnist/1:data`. This code segment means that the dataset available at `https://www.floydhub.com/wish1104/datasets/emnist/1` will be mounted at  a folder called `/data` under the linux root.

 You can also mount the outputs of a previous job like this `--data wish1104/projects/character-recognition/57/output:previous-output`. In this example, mounted at the folder called `/previous-output` is the output of job `57` of a project called `character-recognition` of a FloydHub user named `wish1104`.

### Messages

A meaningful description can make it easy to understand what each job is about. To supply a description, you need to use the `--message '<YOUR_TEXT_MESSAGE>'`, where `<YOUR_TEXT_MESSAGE>` is a description message for the job. Note that the message must be surrounded by quotes. For example, `--message 'hypertuning for 50 epochs'` will add to the job a description "hypertuning for 50 epochs". 

The description, whether supplied or not, can  always be edited on your job's webpage.

### Run-commands

At the end of `floyd run ...` command is a string, surrounded by quotes, indicating what (linux) commands to be run. For example, if you run `floyd run 'pwd'`, a job will be launched and the command `pwd` (print working directory) will be run. 

More often, you want to run a python script with additional arguments. Usage of the `argparse` library is recommended. Suppose you have the following file named `test.py` that reads:

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

However, the output of the job is not directly rendered to your terminal. To read your output and monitor you jobs, refer to the next part of this tutorial.

## Monitor your jobs

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

Here, you will see several tabs `Overview`, `Files`,`Input Data`, `Settings` and possibly `Output`.

### The Overview tab

In the `Overview` tab, you will see a label at the top right corner of the webpage (just under your profile image), which says `Queued`, `Success`, `Running`, ` Shutdown` or `Failed`, indicating the current status of the job. 

You can also see the **system metrics** (collected every minute) about the utilization of CPU, GPU and SSD.

Meanwhile, **training metrics** is automatically parsed from the `stdout` of your job. For a metric to be parsed, write each metric entry in json format like this:

```python
print('{"metric": "<metric_name>", "value": <int_or_float>, "epoch": <int>}'). 
```

**IMPORTANT: It is imperative to print your metrics for every few epochs so that we can assess your model performance. Some metrics to be included are loss/cost, accuracy, precision, recall, f-score (if pertinent) for both training and validation phase** (Detailed instructions [here](https://docs.floydhub.com/guides/jobs/metrics/).)

Also, the `stdout` (e.g. everything you `print()` in python) will be displayed in the `logs` section, as you can see a black text field that resembles a terminal.

### The Files tab & the Input Data tab

In the files tab is a copy of your local project folder (at the time a job is run). Not surprisingly, files that are included in the `.floydignore` file are not inlcuded.

In the Input Data tab is a description of all datasets mounted for the job.

### The Output tab

This tab may not appear on the webpage if you have no output files, or output files whose total size is smaller than 20 KiB. If the tab does not appear, you can still go to it by appending `output` to you job's URL. For example, even though the job `https://www.floydhub.com/wish1104/projects/character-recognition/58/` has no "Output" tab on its web page, you can go to `https://www.floydhub.com/wish1104/projects/character-recognition/58/output` to see its output.

__For each FloydHub instance, only outputs in the /output folder is collected. Therefore, you must always save output files to the /output folder. Otherwise, output files will not be saved or a PermissionError will occur.__

### Tensorboard [Optional]

While a job is still running, a link to a tensorboard server should appear on the job's webpage. If you know how to use tensorboard (or `tensorboardX` for PyTorch) you can instantiate `SummaryWriter`s to save summaries in the `output` folder. 

When the job terminates (with state failed/sucess/shutdown), the tensorboard server is also shutdown. If you want to analyze your metrics, either download the summary files in the Output tab. Alternatively, start a new job (on a CPU instance for cost-efficiency), mount the output of the orignal job, `cp` all summary files to `/output` folder of the new job, and hang the job indefinitely. You can manually shutdown the job once you are finished analyzing the metrics.