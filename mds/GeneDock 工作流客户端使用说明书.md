# GeneDock工作流客户端使用说明书
## 前言

GeneDock允许用户在平台上自行搭建自己的工作流。

本文档旨在向用户说明如何使用GeneDock工作流客户端配置工作流。

### 申明
1. 以下命令示例使用的为Ubuntu版本的客户端，windows用户使用时请注意使用`.\genedock_workflow.exe`调用命令；
2. 关于APP和工作流的配置的方法，请参考《GeneDock APP和工作流配置手册》，此处不再赘述。
3. 以下输出示例中只显示成功的示例，运行不成功时会给出详细提示，此处不再赘述，如下所示：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow -f /var/data/nature/workflow/fastqc-workflow-example.yml
The workflow is fails to new!
The error message is: Invalid input, the workflow name is exist!, code 404
```
> 此提示为工作流出现同名

## 初次使用
### 得到程序
GeneDock 工作流客户端暂时使用邮件发送的方式传递。您会得到相应操作系统客户端的tar压缩包，然后您可以使用类似`tar -xvf genedock_centos_workflow.tar`(windows 可使用360压缩等程序)对文件进行解压缩，解压完后，得到`genedock_workflow`可执行文件（windows为`genedock_workflow.exe`）。
### 获取您的access_key_id和access_key_secret
1. 点击[网站](https://genedock.com)右上角的账户名, 比如"_xxxx@genedock.com_", 在下拉菜单中选择点击"**设置**";
2. 进入到账户设置页面，选"**AccessKeys**";
3. "access_key_id"的值即为您的access_key_id, "access_key_secret"的值即为您的access_key_id所对应的access_key_secret (为了安全考虑, access_key_secret的值不会直接显示，您可以点击"**显示**"就可以看到)
4. 为了方便您进行使用，双击"access_key_id"和"access_key_secret"对应的值即可实现**复制**。

### 第一条命令
无论我们将运行什么命令，为了获得帮助，我们都可以运行`./genedock_worflow`，获得如下帮助信息：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow
No commands.
Command List:
  Version:           version
  Configuration:     config [--id=access_id] [--key=access_key]
  new app:           app -f [PATH_TO_LOCAL_APP_CONFIGURATION_FILE]
  list apps:         app_ls [-t app type]
  update app:        app_update [-i app id] [-f PATH_TO_LOCAL_APP_CONFIGURATION_FILE]
  get app:           app_get [-i app id]
  delete app:        app_delete [-i app id]
  new workflow:      workflow [-f PATH_TO_LOCAL_WORKFLOW_CONFIGURATION_FILE]
  list workflows:    workflow_ls [-t workflow type]
  update workflow:   workflow_update [-i workflow id] [-f PATH_TO_LOCAL_WORKFLOW_CONFIGURATION_FILE]
  get workflow:      workflow_get [-i workflow id]
  delete workflow:   workflow_delete [-i workflow id]
  get workflow parameters workflow_param [-i workflow_id]
  set workflow parameters workflow_param_set [-i workflow_id] [-f PATH_TO_LOCAL_PARAMETER_FILE]
  start task:        run [-i workflow id] [-f PATH_TO_PARAMETER_FILE]
  list tasks:        ls
  get task:          get_task [-i task id]
  list jobs info     jobs [-i task id]
  job log            logs [-i task id] [-j job id]
  job log signature  log_sign [-i task id] [-j job id]
  push image         push [-I repository:tag or image id] [-y] [-E stage/product]
  list image         images
  get image          image_get [-i image mongoDB id]
  delete image       image_delete [-i image mongoDB id]
  set docker image meta  image_meta_set [-i image mongoDB id] [-f PATH_TO_LOCAL_META_FILE]

```
###初始配置
GeneDock 工作流客户端在使用过程中需要与GeneDock的API服务进行通信，并且从安全的角度，采用了基于_access\_id_和_access\_key_的签名授权验证方式。因此在正式使用之前需要进行相关的配置。

通过前面得到的_access\_id_和_access\_key_运行命令：`./genedock_worflow config --id <access_id> --key <access_key>`（`--id`即为_access\_id_，`--key`即为_access\_key_），得到如下提示即为成功：

```bash
kwu@2b668d71f8c1:~$ ./genedock_worflow config --id <access_id> --key <access_key>
Your configuration is saved into /home/kwu/.genedock_credentials
```
##APP配置相关命令
###新建APP
GeneDock允许使用自行编写的yaml创建自己的APP。

运行命令`./genedock_worflow app -f PATH_TO_LOCAL_APP_CONFIGURATION_FILE `即可在GeneDock平台上传并新建一个APP（`-f`为本地APP配置文件的位置），得到如下提示，则为运行成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow app -f /var/data/nature/app/fastqc-app.yml
The app template is inserted successful!
The app id is: 56533377f6f4060016b785ec
```
 >注意：运行成功后会得到该APP的唯一的id，该id会在工作流配置文件的编写、APP的更新和删除等方面用到。

###列出APP
运行命令`./genedock_worflow app_ls` 可以得到账户下已有的可用的APP，得到如下提示即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow app_ls
The app's type is not specify. The default type is 'private'. You can use [-t all/public/private]
    to specify the type of app you want to view!
The app is found successful!
- - _id: 56533377f6f4060016b785ec
    account: xxxxxxxx@genedock.com
    alias: A quality control tool for high throughput sequence data
    category: FASTQ processing
    description: '该流程能够通过fastqc程序对输入的reads快速的进行包括碱基质量、
                  GC含量、长度分布、序列重复、接头和Kmer等一系列的统计，并且生成
                  相应的报告。该流程输入文件为多个fastq文件，程序会对每个文件进行
                  质量检测，并且会在页面显示其中一个的文件的检测报告；输出文件为tgz
                  格式的压缩文件，其中包含所有输入文件的fastqc的输出。'
    homepage: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
    modified_time: '2015-11-23 23:40:39'
    modified_ts: 1448293239.083279
    name: FastQC
    package: fastqc
    status: checked
    type: private
    version: 1
```
> 注意：
>
- `app_ls`命令有`-t`的参数，该参数允许用户根据APP的类型选择输出APP，该参数包括`all``public ``private`三个值可选，默认为`private`；
- `public`是列出公共的一些APP；`private`是列出您的私有APP；`all`是列出两种类型的所有APP，还包括一些系统的APP，load data、store data、mapinput、mapoutput等APP。

####相关字段解释
这里解释列出的APP的一些系统添加的字段：

- `- - _id:`即为APP的唯一id；
- `modified_time: `和`modified_ts: `分别是APP修改的时间和时间戳；
- `status: `为该APP的状态，`checked `表示已经过审核；
- `type:`为APP的类型；

###更新APP
GeneDock允许用户在已有APP的情况下更新APP。
运行命令`./genedock_worflow app_update -i <app_id> -f PATH_TO_LOCAL_APP_CONFIGURATION_FILE`即可对APP进行更新（`-i`为已有APP的id，`-f`为本地APP配置文件的位置），得到如下提示即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow app_update -i 56533377f6f4060016b785ec -f \
                     /var/data/nature/app/fastqc-app.yml
The app is update successfully!
The new app id is: 56533377f6f4060016b785ec
```
>注意：我们默认是对app进行替换。但是当配置文件的APP名字更改时，我们会新建一个APP，并得到一个新的APP id。

###得到一个APP的详细信息
运行命令`./genedock_worflow app_get -i <app_id>` 即可获得该APP的详细信息（`-i`为已有APP的id），输出结果如下所示：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow app_get -i 56533377f6f4060016b785ec
The app info is got successful!
_id: 56533377f6f4060016b785ec
account: xxxxx@genedock.com
alias: A quality control tool for high throughput sequence data
category: FASTQ processing
cmd_template: 'cd /var/data ; mkdir {{ outputs.tgz[0].enid }} ; {{ ''\n'' }} fastqc
  {{ parameters.threads }} -o /var/data/{{ outputs.tgz[0].enid }} {% for read in inputs.reads
  %} {{ read.path }} {% endfor %}; {{ ''\n'' }} if [ $? -ne 0 ]; then {{ ''\n'' }}
  {{ ''\t'' }}echo "the job of fastqc is fail!"; exit 1 {{ ''\n'' }} fi {{ ''\n''
  }} tar zcvf {{ outputs.tgz[0].path }} {{ outputs.tgz[0].enid }} ;{{ ''\n'' }} fastqc=`ls
  /var/data/{{ outputs.tgz[0].enid }}/*.zip | head -n 1 ` ;{{ ''\n'' }} cp $fastqc
  ./ ;{{ ''\n'' }} fastqc_zip=`ls *.zip`; unzip -o $fastqc_zip ; fastqc_out=`ls -d
  *_fastqc`; python /bioapp/report/FastQC.py --output=/var/data/$fastqc_out ;{{ ''\n''
  }}'
description: '该流程能够通过fastqc程序对输入的reads快速的进行包括碱基质量、GC含量、长度分布、序列重复、
              接头和Kmer等一系列的统计，并且生成相应的报告。该流程输入文件为多个fastq文件，程序会对每个
              文件进行质量检测，并且会在页面显示其中一个的文件的检测报告；输出文件为tgz格式的压缩文件，其
              中包含所有输入文件的fastqc的输出。'

..........
..........

```
###删除APP
运行命令`./genedock_worflow app_delete -i <app_id>`即可删除APP（`-i`为需删除APP的id），得到如下提示则删除成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow app_delete -i 56533377f6f4060016b785ec
The app is deleted successful!
```
##工作流配置相关命令
###新建工作流
运行命令`./genedock_worflow workflow -f PATH_TO_LOCAL_WORKFLOW_CONFIGURATION_FILE`即可新建工作流（`-f`为本地的工作流配置文件）并得到唯一的id，得到如下输出即为成功：

```yaml
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow -f /var/data/nature/workflow/fastqc-workflow-example.yml
The workflow is saved successful!
The workflow id is: 5653ef32f6f4060019b48e08
The workflow parameters is:
Conditions:
  schedule: ''
Inputs:
  randomcode0_loaddata:
    alias: raw read
    category: loaddata
    data:
    - enid: <Please input the enid of the data in here>
      name: <Please input the name of the data in here>

...........
...........

```
>注意：成功新建工作流后除了会显示以上信息外，还会在当前目录新建一个`workflow id+_parameters.yaml`的参数模版文件，如上即可得到`5653ef32f6f4060019b48e08_parameters.yaml`，该文件内容即为标准输出的`The workflow parameters is:`之下的部分。关于参数文件的详细介绍请参看附录《模版参数文件的介绍》

###列出工作流
运行命令`./genedock_worflow workflow_ls`即可列出账号下所有的工作流，得到如下输出即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow_ls
The workflow's type is not specify. The default type is 'private'. You can use [-t all/public/private]
    to specify the type of workflow you want to view!
The workflow is found successful!
- _id: 5653ef32f6f4060019b48e08
  account: wukai199010@163.com
  authtype:
    owner: true
    run: true
  description: '该流程能够通过fastqc程序对输入的reads快速的进行包括碱基质量、GC含量、长度分布、序列重复、
               接头和Kmer等一系列的统计，并且生成相应的报告。该流程输入文件为多个fastq文件，程序会对每个文
               件进行质量检测，并且会在页面显示其中一个的文件的检测报告；输出文件为tgz格式的压缩文件，其中包
               含所有输入文件的fastqc的输出。'
  modified_time: '2015-11-24 13:01:38'
  modified_ts: 1448341298.585556
  name: FastQC
  new: true
  status: checked
  type: private
  version: 1

```
>注意：
>
>1. 与APP的list类似，工作流的list也可以选择`-t`参数。
>2. 与APP类似，系统也添加了`type: `等字段，此外，还添加了`authtype: `的字段，`owner: true`表示您是创建者，`run: true`表示您有运行该工作流的权限。

###更新工作流
运行命令`./genedock_worflow workflow_update -i <workflow_id> -f PATH_TO_LOCAL_WORKFLOW_CONFIGURATION_FILE `即可更新工作流（`-i`为待更新的工作流的id，`-f`为本地的工作流配置文件），得到如下输出则为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow_update -i 5653ef32f6f4060019b48e08 \
                    -f /var/data/nature/workflow/fastqc-workflow-example.yml
The workflow is updated successfully!
The new workflow id is: 5653ef32f6f4060019b48e08
The new workflow parameters is:
Conditions:
  schedule: ''
Inputs:
  randomcode0_loaddata:
    alias: raw read
    category: loaddata
    data:
    - enid: <Please input the enid of the data in here>
      name: <Please input the name of the data in here>

.........
.........
```
> 注意：更新工作流时会在当前目录生成新的参数文件；如果工作流的`name:`字段修改，会新生成一个工作流并得到一个新的id。

###得到一个工作流的详细信息
运行命令`./genedock_worflow workflow_get -i <workflow_id>`可以得到工作流的详细信息（`-i`为工作流的id），得到如下提示即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow_get -i 5653ef32f6f4060019b48e08
The workflow is found successful!
_id: 5653ef32f6f4060019b48e08
account: xxxxx@genedock.com
description: '该流程能够通过fastqc程序对输入的reads快速的进行包括碱基质量、GC含量、长度分布、
             序列重复、接头和Kmer等一系列的统计，并且生成相应的报告。该流程输入文件为多个fastq
             文件，程序会对每个文件进行质量检测，并且会在页面显示其中一个的文件的检测报告；输出文
             件为tgz格式的压缩文件，其中包含所有输入文件的fastqc的输出。'
history:
- alias: raw read
  app_id: 55128c58f6f4067d63b956b5
  node_id: randomcode0_loaddata
  outputs:
    data:
      enid: enid_reads
- alias: FastQC analyse
  app_id: 5653eee6f6f4060017b48e08
  inputs:
    reads:
    - enid: enid_reads
  node_id: randomcode1_fastqc
  outputs:
    tgz:
    - enid: fastqc_output
............
............
```

###删除工作流
运行命令`./genedock_worflow workflow_delete -i <workflow_id>`即可删除工作流（`-i`为待删除工作流的id），得到以下提示即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow_delete -i 56541748f6f406000f3f8b0c
The workflow is deleted successful!
```
###得到参数文件
除了新建和更新工作流时会得到模版参数文件，同时也可以运行命令`./genedock_worflow workflow_param -i <workflow_id>`得到相应工作流的模版参数文件（`-i`为工作流的id），文件保存在当前目录下。得到以下提示即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow_param -i 5653ef32f6f4060019b48e08
The workflow parameters is got successful!
```

###为工作流添设置相应的默认参数
GeneDock允许用户提前设置工作流的默认参数和输入，可以是部分，也可以是全部。
当相应的参数文件填好后（此文件不必填写全），运行`./genedock_worflow workflow_param_set -i <workflow_id> -f PATH_TO_LOCAL_PARAMETER_FILE`即可为工作流设置默认参数（`-i`为工作流 id，`-f`为本地参数文件位置），得到如下提示即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow workflow_param_set -i 5653ef32f6f4060019b48e08 \
                    -f 5653ef32f6f4060019b48e08_parameters.yaml
The workflow parameters is set successful!
```

##运行工作流的相关命令
###运行任务
运行任务前请确认您的参数文件是可用的。
运行命令`./genedock_worflow run -i <workflow_id> -f PATH_TO_PARAMETER_FILE`即可运行一个任务（`-i`为工作流的id，`-f`为参数文件），如果成功，会得到以下提示：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow run -i 55c332d5534680619c54bd63 -f \
                     /var/data/nature/product_parameter/parameter/Fastqc_param.yml
The plan is generated successful! The plan is: 565420005346800010081f75.
    The total data size is 0. The cost is: 0

The plan is generated successful. if you want to start the task please input: (yes/y):
```
此提示plan已经成功生成，是否启动运行任务，输入`yes/y`即可启动运行任务。

如果任务启动成功，会得到以下提示：

```
The task is started successful! The task id: 56542002534680001593247b
```
>注意：运行后会得到一个任务的唯一的id，此id将在查看详情、运行日志等命令时用到。

###列出任务
运行`./genedock_worflow ls`命令即可显示账号下所有的任务，将显示各任务的状态和job的个数等，输出如下所示：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow ls
The tasks has been found successfully!
- _id: 55c3082d534680624423687f
  account: xxxxx@genedock.com
  description: FastQC_2015_08_6_15_10_02
  job_number: 1
  modified_time: '2015-08-06 15:09:33'
  modified_ts: 1438844973.920944
  name: FastQC
  progress_rate: 0/1
  start_ts: 1438844973.925731
  status: running
  successful_job_number: 0
  type: private
```

###查看任务详情
运行`./genedock_worflow get_task -i <task_id>`即可获得相应task的详细信息（`-i`为任务id），输出信息如下：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow get_task -i 56542002534680001593247b
The task information is found!
account: wukai@genedock.com
description: fastqc task
end_ts: 1448353852.573662
modified_time: '2015-11-24 16:29:54'
modified_ts: 1448353794.821549
name: fastqc task
parameters:
  Conditions:
    schedule: ''
  Inputs:
    randomcode0_loaddata:
      alias: 输入reads文件
      category: loaddata
.........
.........
```
###查看任务的job信息
运行`./genedock_worflow jobs -i <task_id>`即可查看任务的job信息（`-i`为任务id），输出如下所示：

```bash
kwu@2b668d71f8c1:~$ ./genedock_worflow jobs -i 56542002534680001593247b
The jobs information is got successful!
- alias: FastQC analyse
  app_name: FastQC
  cmd:
 mkdir 565420025346800010aa72a6 ;
 fastqc -t 2 -o /var/data/565420025346800010aa72a6  /var/data/54b7b203534680466ca85e62.fastq ;
 if [ $? -ne 0 ]; then
        echo "the job of fastqc is fail!"; exit 1
 fi
 tar zcvf /var/data/565420025346800010aa72a6.tgz 565420025346800010aa72a6 ;
 fastqc=`ls /var/data/565420025346800010aa72a6/*.zip | head -n 1 ` ;
 cp $fastqc ./ ;
 fastqc_zip=`ls *.zip`; unzip $fastqc_zip ; fastqc_out=`ls -d *_fastqc`;
 python /bioapp/report/FastQC.py --output=/var/data/$fastqc_out ;
  end_time: 1448353852.15
  job_id: 56542002534680001593247b_565420005346800010081f75_randomcode1_fastqc
  start_time: 1448353825.31
```
####字段详解
- `cmd:`为APP配置文件的`cmd_template`经过参数和输入输出的替换后生成的shell script；
- `job_id:`为每个job的唯一的id；
- `status:`为job的状态

###查看job的运行日志
运行命令`./genedock_worflow logs -i <task_id> -j <job_id>`即可查看job的运行日志（`-i`为相应的任务id，`-j`为相应的job id），得到如下输出：

```
wu@2b668d71f8c1:~$ ./genedock_worflow logs -i 56542002534680001593247b \
                  -j 56542002534680001593247b_565420005346800010081f75_randomcode1_fastqc
The job's log is got successful!
Started analysis of 54b7b203534680466ca85e62.fastq
Approx 5% complete for 54b7b203534680466ca85e62.fastq
Approx 10% complete for 54b7b203534680466ca85e62.fastq
Approx 15% complete for 54b7b203534680466ca85e62.fastq
Approx 20% complete for 54b7b203534680466ca85e62.fastq
Approx 25% complete for 54b7b203534680466ca85e62.fastq
Approx 30% complete for 54b7b203534680466ca85e62.fastq
Approx 35% complete for 54b7b203534680466ca85e62.fastq
Approx 40% complete for 54b7b203534680466ca85e62.fastq
Approx 45% complete for 54b7b203534680466ca85e62.fastq
Approx 50% complete for 54b7b203534680466ca85e62.fastq
Approx 55% complete for 54b7b203534680466ca85e62.fastq
Approx 60% complete for 54b7b203534680466ca85e62.fastq
Approx 65% complete for 54b7b203534680466ca85e62.fastq
Approx 70% complete for 54b7b203534680466ca85e62.fastq

.............
.............
```
> 注意：实时查看job的日志只有200行，超过的可以通过下载文件的形式获取。

###得到job运行日志文件
GeneDock允许用户通过获取日志链接的方式下载日志文件。

运行命令`./genedock_worflow log_sign -i <task_id> -j <job_id>`即可获得日志链接（`-i`为相应的任务id，`-j`为相应的job id），该链接通过浏览器可下载。得到如下输出即为成功：

```
kwu@2b668d71f8c1:~$ ./genedock_worflow log_sign -i 56542002534680001593247b -j \
                    56542002534680001593247b_565420005346800010081f75_randomcode1_fastqc
The job log's signature is got successful!
http://job-log.oss-cn-beijing.aliyuncs.com/56542002534680001593247b \
%2F56542002534680001593247b_565420005346800010081f75_randomcode1_fastqc.log? \
OSSAccessKeyId=juCjgae1f42vSAv4&Expires=1448358084&Signature=1KsZyGuhy3xYqSZ7yMIDZa4aE2E%3D
```

###Image相关命令
iamge相关命令暂不开放，尽情期待！

如有上传image的需求，请联系GeneDock!

##附录

###模版参数文件的介绍
在成功新建一个工作流后，程序会返回一个参数文件，该参数文件将在运行工作流时提供具体的参数，下面将对参数文件进行详细的解释。
####模版示例
```yaml
Conditions:           
  schedule: ''
Inputs:
  randomcode0_loaddata:
    alias: raw read
    category: loaddata
    data:
    - enid: <Please input the enid of the data in here>
      name: <Please input the name of the data in here>
      property:
        block_file:
          block_name: null
          is_block: false
          split_format: default
    formats:
    - fq
    - fastq
    - gz
    maxitems: 100
    minitems: 1
    required: true
    type: file
Outputs:
  randomcode2_storedata:
    alias: store fastqc zip output
    data:
    - description: <Please input the description of the output data in here>
      name: <Please input the name of the output data in here>
      property:
        block_file:
          block_name: null
          is_block: false
          split_format: default
    formats:
    - tgz
    maxitems: 1
    minitems: 1
    type: file
Parameters:
  randomcode1_fastqc:
    alias: FastQC analyse
    parameters:
      threads:
        hint: Specifies the number of files which can be processed simultaneously.  Each
          thread will be allocated 250MB of memory so you shouldn't run more threads
          than your available memory will cope with, and not more than 6 threads on
          a 32 bit machine
        maxvalue: 16
        minvalue: 1
        required: true
        type: integer
        value: 2
        variable: true
Property:
  CDN:
    required: true
  reference_task:
  - id: <Please input the reference task's id>
  water_mark:
    required: true
    style: null
description: <Please input the task's description in here>
name: <Please input the task's name in here>

```
####字段详细解释
1. `Conditions: `为调度相关，暂时不开放，不用做任何动作；
2. `Inputs:`为填写相关输入数据的信息；
 - `randomcode0_loaddata:`为输入数据的node id；
 - `data:`字段为填写该位置输入数据的array（数组的个数由`minitems `和`maxitems`决定），以下代码为一个单元，多输入时需有多个对应的单元：

     ```
        - enid: <Please input the enid of the data in here>
          name: <Please input the name of the data in here>
          property:
            block_file:
              block_name: null
              is_block: false
              split_format: default
     ```
 - `enid:`为数据的enid；
 - `name:`为数据的名字；
 - `property:`为数据的保留字段，在流式上传中使用，普通上传时不需要修改。

      >注意：
      >
      > - 以上字段可以修改，其他字段为参考字段，如`maxitems: 100`等为参考的字段，不能修改。
      > - 如果输入`required:`为`false`且确实没有输入，不做任何改动即可，`<Please input the enid of the data in here>`等也不要删除。
3. `Outputs:`为填写输出数据的相关信息；
 - 与 `Inputs:`相似，只是输出数据没有`enid`，而是填写数据的描述信息`description:`
 - `name:`和`description:`不能为空或不做修改

     >注意：运行工作流时，会自动覆盖同名的输出文件，注意修改。
4. `Parameters:`为修改参数字段；
 - 不同的运行node会有不同的参数，填写相关参数的`value:`字段即可，其他字段为参考字段不需要修改。

     >注意：各参数的值需符合该参数对应的描述，如类型，值的范围等等。
5. `Property:`为图片水印或参考task的选项；
 - `reference_task:`为参考task的选项，此选项暂不提供，请将`_id:`的值改为`null`
 - `water_mark:`为图片水印选项，`required: false`为去水印
 - 其他字段暂不需修改
6. `description:`为该task任务的描述，可以不做修改；
7. `name: `为该task任务的名字，可以不做修改。

###深圳域的使用方法
当前目录下新建一个隐藏文件夹（文件夹名称：`.gdconfig`)，然后在文件夹创建一个名为`.gdenvironment.yml`的隐藏yaml文件，文件内容为：

```yaml
GD_PROTOCOL: "https"
GD_HOST: "shenzhen.genedock.com"
GD_PORT: "443"
```
其中 `GD_PROTOCOL` 为使用的协议（即：https 或者 http）， `GD_HOST` 为api server 的host， `GD_PORT`为 api server 的监听端口，以上参数可以根据实际情况进行修改。


##FAQ
1. 出现以下错误，返回的提示并没有什么有用信息，原因是什么？该怎么办？

  ```
The error message is: server internal error!
  ```
答：出现以上问题，首先检查自己的access\_id和access\_key是否正确，如果还是出错，可以找GeneDock Support支持。

2. 以下问题是什么原因？

  ```
kwu@2b668d71f8c1:~$ ./genedock_worflow ls
NetWork Connection Error, Retrying...
  ```
答：这个问题一般出现在无法联网或者`.gdenvironment.yml`配置错误.
