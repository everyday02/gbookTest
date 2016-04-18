#怎样在GeneDock配置一个_GATK Best Practices_工作流
##前言
本文档旨在作为示例，指导用户在GeneDock平台配置自己的工作流。
###申明
1. 流程参考《[GATK Best Practices](https://www.broadinstitute.org/gatk/guide/bp_step.php?p=1)》，但是做了调整，如：使用`hardFilters`代替`VariantFiltration ＋ VariantRecalibrator ＋ ApplyRecalibration`；使用`samtools sort`代替`picard sort`，等；
2. 流程中各程序的参数只列出常见的部分，其他参数可自行添加。

###流程介绍
如下图所示，本流程以一对`fastq`文件作为输入，经过`bwa mem` mapping后得到bam文件，然后用`picard MarkDuplicates`标记重复，接着经过GATK预处理后，输出的bam用`GATK HaplotypeCaller`寻找突变位点，通过`GATK hardFilters`对突变位点进行筛选后得到最终的输出文件。

![](http://staging-reference.oss-cn-beijing.aliyuncs.com/example_data%2Fkwu_test%2Finfo.png?OSSAccessKeyId=f8J26MSNAHZezrlF&Expires=1485465491&Signature=jqq6ssMXGurap%2F5uUIcANiBHmi0%3D)

###编写Dockerfile并得到image的id
请参考《创建一个能够在GeneDock平台运行的Docker镜像》里面的附录的Dockerfile.

通过客户端上传`genedock/gatk:latest`镜像后得到image的id`565831b853468000127de27c`
>注意：
>
>1. 现在暂时不支持用户自行上传docker镜像，如需上传，请联系GeneDock！

###APP的配置
关于APP的配置，这里将详细介绍`bwa mem`（附件`bwa_mem_app.yml`）的配置。其他APP的配置，请参考附件的yaml文件。
####bwa mem命令示例
以下是需要配置的`bwa mem`的命令示例，需要注意的是，为了节省时间和资源，我们直接对`bwa mem`的输出结果直接进行格式转换和排序。

```
 bwa mem -M -t 4 -R '@RG\tID:id\tSM:sm\tPL:illumina\tLB:lb' -k 19 \
  /var/data/refgenome/genome.fa /var/data/reads1.fastq.gz \
  /var/data/reads2.fastq.gz | samtools view -@ 4 -b -S - \
  | samtools sort -@ 4 - /var/data/bwa_out/output.sorted 
```
####配置APP基本信息
得到命令后，我们首先配置APP的基本信息，如名字、分类、软件主页、版本描述和基本命令示例等。具体如下所示：

```yaml
app:
  package: bwa
  category: "序列比对工具"
  homepage: http://bio-bwa.sourceforge.net/
  version: 1
  name: bwa-mem-tmp
  alias: Align 70bp-1Mbp query sequences with the BWA-MEM algorithm.
  description: >
    The BWA-MEM algorithm performs local alignment. It may produce 
    multiple primary alignments for different part of a query sequence. 
    This is a crucial feature for long sequences. However, some tools 
    such as Picard’s markDuplicates does not work with split alignments. 
    One may consider to use option -M to flag shorter split hits as secondary.
  tutorial: >
    bwa mem ref.fa read1.fq read2.fq > aln-pe.sam
  document_author: xxxxxxx@genedock.com
```
>注意：
>
>1. `name`字段必须保持唯一，但是可以有多个版本。
>2. 具体分类请参考《GeneDock APP和工作流配置手册》

####配置APP运行环境
配置APP的基本信息后，如下所示我们需要配置APP运行的环境，设置APP运行时所需的CPU、内存和磁盘空间等。

```yaml
  requirements:
    container:
      type: docker
      image: 565831b853468000127de27c
    resources:
      cpu: 4
      mem: 16384m
      network: true
      port: []
      disk: 16384m
```
>注意：
>
>1. 我们作业全部运行在容器中；现在暂时只支持docker的容器；
>2. `image`字段为上传image后得到的image的唯一id（现在image的上传只能由GeneDock的人员代为上传）；
>3. 资源请按实际的运行最大资源填写，docker容器运行时的mem是不能超出请求的mem的；
>4. mem和disk的单位为MB(简写为m)。

####配置APP的输入
`bwa mem` 的输入文件有三个，分别为参考基因组及其相应的bwa索引、一对双端的reads。如下所示：

```yaml
  inputs:
    refgenome:
      hint: index files built by bwa index
      type: file
      required: true
      minitems: 1
      maxitems: 1
      item:
        separator:
      formats: ['tgz']
    reads:
      hint: 5' read sequence
      type: file
      required: true
      minitems: 1
      maxitems: 1
      item:
        separator:
      formats: ['gz', 'fq', 'fastq']
    mates:
      hint: 3' read sequence
      type: file
      required: true
      minitems: 1
      maxitems: 1
      item:
        separator:
      formats: ['gz', 'fq', 'fastq']
```
>注意：
>
>1. `refgenome`为tgz格式，是因为GeneDock暂不支持文件夹的格式，所以先要将基因组文件及其相应的索引文件打包成tgz文件；
>2. reads的格式可支持gz、fq、fastq，为数组类型；
>3. 输入数据个数的最大和最小值可调整，多输入的前提是您使用的程序确实接受多输入；
>4. 其他字段的解释详见《GeneDock APP和工作流配置手册》。

####配置APP输出
APP的输出的配置方式与输入类似，`bwa mem`的输出为bam文件，如下所示：

```yaml
  outputs:
    bam:
      hint: Aligned sequcnes in bam format
      type: file
      required: true
      minitems: 1
      maxitems: 1
      item:
        separator:
      formats: 'bam'
```
>注意：
>
>1. 每个输出项的最大值和最小值均只能为1，不能为多输出，但可以多输出项。
>2. 输出项的格式只能为一个，所以与输入不同，类型为字符；
>3. 其他字段的解释详见《GeneDock APP和工作流配置手册》。

####配置APP参数
`bwa mem`使用了`-M`、`-R`、`-k`参数，依次为`flag`、`string`、`number`类型，配置如下所示：

```yaml
  parameters:
    mark_short_split_hits_as_secondary:
      separator: ' '
      prefix: -M
      required: false
      type: flag
      default: true
      hint: Mark shorter split hits as secondary (for Picard compatibility)
    read_group:
      separator: ' '
      prefix: -R
      required: false
      type: string
      default: ' '
      hint: Complete read group header line. ’\t’ can be used in STR and will be converted to a TAB in the output SAM. The read group ID will be attached to every read in the output. An example is ’@RG\tID:foo\tSM:bar’. 
    minimum_seed_length:
      separator: ' '
      prefix: '-k'
      required: false
      type: number
      default: 19
      hint: 'Minimum seed length. Matches shorter than INT will be missed. The alignment speed is usually insensitive to this value unless it significantly deviates 20. [19]' 
```
>注意：
>
>1. `hint`的值会作为提示在网页显示；
>2. 当`type: string`时，默认`quotes: true`，如想去掉引号，可以添加`quotes: false`
>3. 关于参数的详细信息，请参考《GeneDock APP和工作流配置手册》

####配置APP命令模版
配置完参数后可以通过配置命令模版来实现需要运行的程序，如下所示，将bwa mem的命令通过jinja2模版实现，

```yaml
  cmd_template: >
    mkdir /var/data/refgenome ;{{ '\n' }}
    tar xzvf {% for ref in inputs.refgenome %} {{ ref.path }} {% endfor %} 
    -C /var/data/refgenome ; {{ '\n' }}
    BWT=$(ls /var/data/refgenome/*bwt);{{ '\n' }}
    INDEX=$(basename $BWT .bwt);{{ '\n' }}
    mkdir /var/data/bwa_out ;{{ '\n' }}
    bwa mem {{parameters.mark_short_split_hits_as_secondary}} 
    -t {{ requirements.resources.cpu }} 
    {{parameters.read_group}} {{ parameters.minimum_seed_length }} 
    /var/data/refgenome{{ '/$INDEX' }} 
    {% for read in inputs.reads %}{{ read.path }}{% endfor %} 
    {% for mate in inputs.mates %}{{ mate.path }}{% endfor %} | 
    samtools view -@ {{ requirements.resources.cpu }} -b -S - | 
    samtools sort -@ {{ requirements.resources.cpu }} - 
    /var/data/bwa_out/output.sorted ;{{ '\n' }}
    mv /var/data/bwa_out/output.sorted.bam 
    {% for output_bam in outputs.bam %} {{ output_bam.path }} {% endfor %} 
    ;{{ '\n' }}
```
>注意：
>
>1. APP命令模版到实际运行的shell script是经过jinja2替换后生成的关于jinja2的介绍，可以参考[jinja2](http://jinja.pocoo.org/)；
>2. 详细的解析方式可参考《GeneDock APP和工作流配置手册》； 
>3. 由于数据的基因组及其index是tgz打包后的文件，所以首先需要通过shell script拿到基因组文件的名字；
>4. `bwa mem`的最后输出需要重命名为标准的输出项；
>5. 输入和输出的名字都会被相应的id的替换，`path`不是以原名替换，而是`/var/data/ + enid + format`。

####上传APP配置文件
APP配置文件的上传通过GeneDock的工作流客户端进行上传，具体工作流客户端的使用请参考《GeneDock 工作流客户端使用说明书》。如下所示，我们将`bwa mem`的配置文件上传至GeneDock平台，并得到相应的APP id。

```
$ ./genedock_workflow app -f bwa_mem_app.yml
The app template is inserted successful!
The app id is: 565aaff05346800017082034
```
>注意：
>
>1. 上传配置文件得到的APP id将会在更新APP或配置工作流时用到。

###工作流的配置
如下图所示，整个工作流的输入与输出逻辑，将各APP串联成一个工作流。每个APP在工作流中作为一个节点，我们需要配置每个节点信息。以下将详细介绍配置`bwa mem`的节点信息。工作流的整个配置，见附件`GATK_Template_workflow.yml `

![](http://staging-reference.oss-cn-beijing.aliyuncs.com/example_data%2Fkwu_test%2Fin_out.png?OSSAccessKeyId=f8J26MSNAHZezrlF&Expires=1485465422&Signature=9ofk%2BTTmtJ8OAoDou6I9Y9B7U5k%3D)

>图注：
>
>1. 四边形表示原始输入节点或最终输出节点，方形表示APP节点；
>2. 四边形内文本和方形下方文本对应`GATK_Template_workflow.yml`文件内的enid。

####配置bwa mem的节点信息
节点信息包括其配置文件中唯一的`node_id`、对应的APP的别名、APP id、APP的输入输出和参数。如下所示，为完整的`bwa mem`的node信息：

```yaml
    - node_id: bwa_mem
      app_id: 565aaff05346800017082034
      alias: bwa mem
      inputs: 
        refgenome:
          - enid: index_enid
        reads:
          - enid: read1_enid
        mates:
          - enid: read2_enid
      outputs:
        bam:
          - enid: aligned_bam_enid
      parameters:
        mark_short_split_hits_as_secondary:
          variable: true
          value: true
        read_group:
          variable: true
          value: "@RG\tID:id\tSM:sm\tPL:illumina\tLB:lb"
        minimum_seed_length:
          variable: true
          value: 19
```
>注意：
>
>1. `app_id `为上传APP配置文件后得到的唯一的id；
>2. `node_id `不能与配置文件中其他的node_id重名；
>3. `inputs``outputs``parameters`的键必须与相应的APP配置文件中的键相对应，`required: true`的键必须在配置文件出现；
>4. `inputs`中出现的enid必须是其他节点（包括`loaddata`节点和普通节点）的输出；
>5. 具体的详细的解释请参考《GeneDock APP和工作流配置手册》

####上传工作流配置文件
使用工作流客户端上传工作流的配置文件，具体工作流客户端的使用请参考《GeneDock 工作流客户端使用说明书》。如下所示，会得到工作流的id和相应的参数文件：

```
$ ./genedock_workflow workflow -f GATK_Template_workflow.yml
The workflow is saved successful!
The workflow id is: 565fff315346800015082006
The workflow parameters is:
Conditions:
  schedule: ''
Inputs:
  load_bed_file:
    alias: 目标区域文件
    category: loaddata
    data:
 ..........
 ..........
```
>注意：
>
>1. workflow id将在更新工作流时使用。

###附录
附件包含一下文件：

```
.
├── GATK_BaseRecalibrator_app.yml
├── GATK_HaplotypeCaller_app.yml
├── GATK_IndelRealigner_app.yml
├── GATK_PrintReads_app.yml
├── GATK_RealignerTargetCreator_app.yml
├── GATK_Template_workflow.yml
├── GATK_hardFilters_app.yml
├── bwa_mem_app.yml
└── picard_MarkDuplicates_app.yml

0 directories, 9 files
```