# GeneDock report配置手册

##生成报告内容目录
**GeneDock**支持用户自定义报告模版。目前报告仅支持网页在线浏览。报告显示页面中所呈现的内容支持csv格式的表格和png格式的图片。
当Workflow中某个APP生成了报告所需要的数据，用户需要将这些数据放到该容器中的指定路径以便于系统识别和替换。默认情况下，APP运行结束后，系统会扫描docker容器内的`/var/data`目录下是否有`report`文件夹，同时，`report`文件夹内需要有`Tables`和`Images`文件夹来分别存放表格和图片。如果有文字内容的替换，记录在`Data.yml`中。
具体文件路径如下：
  
```
/var/data/  
└── report  
    ├── Data.yml
    ├── Images
    └── Tables

``` 

##markdown编写
我们采用标准的[markdown格式](http://wowubuntu.com/markdown/) ，将结果进行替换。若需要手动添加某些固定表格或外链图片，请参见markdown语法。

##结果数据表/图和文字内容的替换
1. 数据图表  
以FastQC报告为例:  
`$tmp_randomcode1_fastqc:Basic Statistics_table$`

其中：
  
- `tmp_`为固定前缀。  
- `randomcode1_fastqc`为workflow配置文件中的node id    
- `Basic Statistics_data`为该node下APP中的对应内容。  
  *如，生成了名为 `Basic Statistics_table.csv` 的统计结果文件，则当键值对`$tmp_randomcode1_fastqc:Basic Statistics_table$` 存在时，系统会自动查找`Tables`和`Images`目录下符合该文件名（不包括后缀）的文件，与该键值对进行替换以显示在页面上。* 

2. 文字内容的替换:  
如果程序本身生成了一些变量需要显示在报告中，可以记录在`Data.yml`中用以替换。  
还是以FastQC报告为例：  
FastQC会根据数据质量情况判断质量是`PASS`或是`FAIL`。  
我们可以将生成的结果以键值对的形式纪录如下，存放在Data.yml中：

```
Adapter Content_data: PASS
Basic Statistics_data: PASS
Kmer Content_data: WARN
Overrepresented sequences_data: PASS
Per base N content_data: PASS
Per base sequence content_data: PASS
Per base sequence quality_data: PASS
Per sequence GC content_data: PASS
Per sequence quality scores_data: PASS 
Per tile sequence quality_data: PASS 
Sequence Duplication Levels_data: PASS 
Sequence Length Distribution_data: PASS
```
例如当报告中需要替换`Basic Statistics_data`的值`PASS`时，在报告模版中加入`$tmp_randomcode1_fastqc:Basic Statistics_data$` 则完成该处替换。

___

**注意：因此，生成报告所需要的图片表格文件名不能重复，即便文件扩展名不同**
___

##实战范例

文件开头的`[TOC]`表示"Table of Content"。添加此行可以按照markdown的标题层级在页面上正确显示报告目录。
以FastQC为例，在任务中生成的report目录如下

```
/var/data/  
└─ report  
   ├── Data.yml  
   ├── Images
   │   ├── adapter_content.png
   │   ├── duplication_levels.png
   │   ├── kmer_profiles.png
   │   ├── per_base_n_content.png
   │   ├── per_base_quality.png
   │   ├── per_base_sequence_content.png
   │   ├── per_sequence_gc_content.png
   │   ├── per_sequence_quality.png
   │   ├── per_tile_quality.png
   │   └── sequence_length_distribution.png
   └── Tables
       ├── Adapter Content_table.csv
       ├── Basic Statistics_table.csv
       ├── Kmer Content_table.csv
       ├── Overrepresented sequences_table.csv
       ├── Per base N content_table.csv
       ├── Per base sequence content_table.csv
       ├── Per base sequence quality_table.csv
       ├── Per sequence GC content_table.csv
       ├── Per sequence quality scores_table.csv
       ├── Per tile sequence quality_table.csv
       ├── Sequence Duplication Levels_table.csv
       └── Sequence Length Distribution_table.csv
```
report配置文件`FastQC.md`如下

```
 [TOC]

 # FastQC Report

 ## 1.Basic Statistics ($tmp_randomcode1_fastqc:Basic Statistics_data$)
 $tmp_randomcode1_fastqc:Basic Statistics_table$

 ## 2.Per base sequence quality ($tmp_randomcode1_fastqc:Per base sequence quality_data$)
 $tmp_randomcode1_fastqc:per_base_quality$
 $tmp_randomcode1_fastqc:Per base sequence quality_table$

 ## 3.Per tile sequence quality

 ## 4.Per sequence quality scores ($tmp_randomcode1_fastqc:Per sequence quality scores_data$)
 $tmp_randomcode1_fastqc:per_sequence_quality$
 $tmp_randomcode1_fastqc:Per sequence quality scores_table$

 ## 5.Per base sequence content ($tmp_randomcode1_fastqc:Per base sequence content_data$)
 $tmp_randomcode1_fastqc:per_base_sequence_content$
 $tmp_randomcode1_fastqc:Per base sequence content_table$

 ## 6.Per sequence GC content ($tmp_randomcode1_fastqc:Per sequence GC content_data$)
 $tmp_randomcode1_fastqc:per_sequence_gc_content$
 $tmp_randomcode1_fastqc:Per sequence GC content_table$

 ## 7.Per base N content ($tmp_randomcode1_fastqc:Per base N content_data$)
 $tmp_randomcode1_fastqc:per_base_n_content$
 $tmp_randomcode1_fastqc:Per base N content_table$

 ## 8.Sequence Length Distribution ($tmp_randomcode1_fastqc:Sequence Length Distribution_data$)
 $tmp_randomcode1_fastqc:Sequence Length Distribution_table$
 $tmp_randomcode1_fastqc:sequence_length_distribution$

 ## 9.Sequence Duplication Levels ($tmp_randomcode1_fastqc:Sequence Duplication Levels_data$)
 $tmp_randomcode1_fastqc:duplication_levels$
 $tmp_randomcode1_fastqc:Total Duplicate Percentage$
 $tmp_randomcode1_fastqc:Sequence Duplication Levels_table$

 ## 10.Overrepresented sequences ($tmp_randomcode1_fastqc:Overrepresented sequences_data$)
 $tmp_randomcode1_fastqc:Overrepresented sequences_table$

 ## 11.Adapter Content

 ## 12.Kmer Content ($tmp_randomcode1_fastqc:Kmer Content_data$)
 $tmp_randomcode1_fastqc:kmer_profiles$
 $tmp_randomcode1_fastqc:Kmer Content_table$
 ```