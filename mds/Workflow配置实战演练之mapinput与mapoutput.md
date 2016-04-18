# Workflow配置实战演练之mapinput与mapoutput


##mapinput与mapoutput
如果说APP内部jinjia2中的for循环提供了容器内文件的批量处理，那么mapinput和mapoutput的使用则可看作是一种APP层面的批量处理方式，而且它们是并行处理的：对这批数据中的每个输入文件启动一个容器来运行指定的APP。

*mapinput和mapoutput示意图*
![](http://ww1.sinaimg.cn/mw690/749452a5gw1ez1cycc88ej20ky05ewey.jpg)


mapinput/mapoutput是 ***GeneDock*** 提供的内置APP，对数据本身不做处理。mapinput和loaddata一样，只读取数据并作为下游APP的输入；而mapoutput只接收上游多个***并行*** 任务的输出，将它们合并到一个指定的输出流中。一个Workflow中mapinput和mapoutput需要成对出现，个数保持相同。

##mapinput详解
####适用范围和使用限制  
mapinput适用于一批相同类型数据需要同时启动多个容器，运行相同后续流程的场景。
当Workflow运行时，根据mapinput输入文件的个数，将会同时启动相应个数的容器来运行下游的APP。同时，mapinput有以下限制：  

- 当使用mapinput代替loaddata作为输入的方式时，`inputs`字段为空。mapinput直接作为一种读取数据（列表）的方式，**不需要**有上游APP的输出作为其输入。因此，mapinput不能作为workflow接收上游结果并分发数据到下游做并行计算的工具。  
- mapinput读取的数据格式统一。一般情况下，mapinput的输入是一个批次的待处理数据，这些数据必须采用同样的下游处理流程。

####position  
如果mapinput下游的第一个APP有两个输入，则需要mapinput提供两个输出。这两个输出我们称作是两个positions。在Workflow配置文件中如下呈现：

```
- node_id: randomcode0_mapinput
  app_id: 55c48f0cf6f40600201d9b98
  alias: 'mapinput two positions'
  inputs: null
  outputs:
    position1:
      enid: "enid_position1"
    position2:
      enid: "enid_position2"
  parameters:
    input_position_number:
      variable: true
      value: 0
    output_position_number:
      variable: true
      value: 2
    output_scatter_method: "dot_product"
```  
可以有更多的position存在，如position3，position4……  
需要注意的是`parameters`中的`output_scatter_method`字段。当值为`dot_product`时，表示各个position中的输入文件会根据列表中的顺序，成对地被分配到下游的APP中。  
**因此，`dot_product`模式的mapinput需要保证每个postion输入的文件个数相同**  

- 假设一个mapinput有两个position: position1和position2，每个position都有3个文件：position1有n1, n2, n3三个输入文件; position2有m1, m2, m3三个输入文件。  
该mapinput下游接有一个APP，该APP有两个输入：input1和input2，input1指向position1，input2指向position2。当实际运行时，将会同时启动三个的APP，其中input1和input2分别为 [n1, m1], [n2, m2], [n3, m3]。

![](http://ww1.sinaimg.cn/mw690/749452a5gw1ez1cy9q034j20d809o74v.jpg)

####叉乘模式  
***GeneDock*** 的mapinput提供mapinput的叉乘模式。如上节所述，当`output_scatter_method`字段为`dot_product`时，需要保证各个postion传入的输入文件个数是一致的。而使用叉乘模式时，该字段为**`cross_product`**, 各个position的输入文件个数可以相同，也可以不同，但不能为0。  

- 假设某个mapinput有两个position，第一个position有n个文件被读取(n >= 1)，第二个position有m个文件被读取(m >= 1)，那么如果下游的APP同时接入position1和position2两个输入，将会有 **n x m** 种组合方式。Workflow运行时将会同时启动 **n x m** 个容器来运行后续的流程。

![](http://ww2.sinaimg.cn/mw690/749452a5gw1ez1cyb25nej20ci0cz3zc.jpg)
  
___
#####注意：从mapinput后直至mapoutput，其间经过的APP除流动的文件外，其它固定文件和参数都应该保持恒定。
___  

##mapoutput详解
#### 适用范围和使用限制  
如果在Workflow中使用了mapinput，则需要有对应的mapoutout对并行流程的计算结果进行收集和定向输出。mapoutput同样不会对数据本身做任何改动，不会对结果文件的内容做合并，而只是从上游各个并行的APP中接收结果。  
mapoutput不会和storedata一样存储结果文件并上传到云端。mapoutput的数据流和普通的APP一样，有inputs也有outputs。因此，在mapoutput的下游还可以继续接APP对这些结果文件进行进一步处理。 

####position 
在mapoutput中，inputs和outputs都以position接收和传出文件。  
如果mapoutput下游接了普通APP，则APP的input的相关信息如`minitems`，`maxitems`，`formats`等也会加入到mapoutput的input当中。

- 如果mapoutput只有一个output的position，则根据input的position传出所有接收到的上游数据。
- 如果上游APP有两个不同格式的输出（假设txt文件和fasta文件），那么mapoutput的inputs用两个position来接收它们（position1接收txt文件；position2接收fasta文件）。下游接两个普通的APP分别对两种格式的数据进行处理，则需要mapoutput有两个output的position做输出。其中，output的position1对应input的position1（只传出所有txt文件），output的position2对应input的position2（只传出所有fasta文件）。
![](http://ww1.sinaimg.cn/mw690/749452a5gw1ez1cybu0csj20i607gglw.jpg)

```
- node_id: randomcode1_mapoutput
  app_id: 55c48f12f6f406001b1d9b95
  alias: 'mapoutput'
  inputs:
    position1:
      enid: "enid_appoutput"
  outputs:
    position1:
      enid: "enid_mapouput"
  parameters:
    input_position_number:
      variable: true
      value: 1
  output_position_number:
      variable: true
      value: 1
```
- 假设上游利用mapinput并行了n个APP，该APP的配置文件中有一个输出项`enid: "enid_mapoutput"`, 则mapoutput的输入将收集所有上游n个APP中`enid`为`"enid_mapoutput"`的数据，并传出到outputs中的position1中，以`enid: "enid_mapouput"`作为输出。

## 实战范例
#####叉乘模式的BLAST工作流
该工作流以多个fasta文件和多个分块数据库文件作为输入，使用叉乘模式的mapinput，将BLAST结果用mapouput收集后进行合并处理，最后输出。

![](http://ww1.sinaimg.cn/mw690/749452a5gw1ez1cyaeyu1j20q108rjs8.jpg)

```
workflow:
  name: 'blastall_cross'
  description: 'blast_cross'
  account: public@genedock.com
  version: 1
  nodelist:
    - node_id: randomcode1_mapinput
      app_id: 55c48f0cf6f40600201d9b98
      alias: 'database map input'
      inputs: null
      outputs:
        position1:
          enid: "enid_query"
        position2:
          enid: "enid_database"
      parameters:
        input_position_number:
          variable: true
          value: 0
        output_position_number:
          variable: true
          value: 2
        output_scatter_method: "cross_product"
    - node_id: randomcode2_blast
      app_id: 5665649ac21f9600114821f6
      alias: 'blast'
      inputs:
        query:
            - enid: "enid_query"
        database:
            - enid: "enid_database"
      outputs:
        output:
            - enid: "enid_blastoutput"
      parameters:
        evalue:
          variable: true
          value: '1e-5'
        format:
          variable: true
          value: 8
        processors:
          variable: true
          value: 8
        program:
          variable: true
          value: 'blastp'
    - node_id: randomcode3_mapoutput
      app_id: 55c48f12f6f406001b1d9b95
      alias: 'blast outputs'
      inputs:
        position1:
          enid: "enid_blastoutput" 
      outputs:
        position1:
          enid: "enid_blast" 
      parameters:
        input_position_number: 
          variable: true
          value: 1
      output_position_number:
          variable: true
          value: 1
    - node_id: randomcode4_merge
      app_id: 566564a5c21f9600144821f7
      alias: 'blast'
      inputs:
        blastresult:
            - enid: "enid_blast"
      outputs:
        output:
            - enid: "enid_output"
      parameters:
        tag:
          variable: true
          value: 'animal'
        key:
          variable: true
          value: 'blastresult'
    - node_id: randomcode5_storedata
      app_id: 55128c94f6f4067d63b956b6
      alias: 'blast result'
      parameters:
        name:
          variable: yes
        description:
          variable: yes
      inputs:
        data:
          enid: "enid_output"

```