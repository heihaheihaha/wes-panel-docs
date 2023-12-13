|修订历史            |修订原因|修订者|
|-------------------|-----------------|-------------|
|2023.10.30|初步文档|wty|
|2023.11.13|添加外链，语句修饰|wty|
|2023.11.14|Step1，查看json|JXH|
|2023.11.14|更新JXH's newer config.yaml|wty|
|2023.11.14|内容扩充|wty|
|2023.11.14|补充说明config设置与介绍|JXH|
|2023.11.15-20|内容扩充|wty|
|2023.11.20|链入FAQ|wty|

# 简介
本流程接受单样本/双样本的WES/WGS/panel/的fastq数据,参照GATK最佳实践([GATK Best practices](https://gatk.broadinstitute.org/hc/en-us/categories/360002302312)),最终生成GVCF([Genomic Variant Call Format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531812) )

最新的文档可以点击[此处](https://github.com/heihaheihaha/Call_variants_V1.0.0/wiki)获取.

WES/panel流程基本一致.<br>
WGS减少了不适配参数和方法的rule的使用.<br>
但使用方式基本一致，整体思路: ①生成sample的json ② 设置config ③配置环境

文档另设[FAQ](https://github.com/heihaheihaha/Call_variants_V1.0.0/wiki/FQA)

# 期望输入
1. 双端测序fastq文件 （后缀`.fq`, `.fq.gz` etc.）
2. bed文件 （后缀 .bed）

# 运行
## step1 ##
使用以下脚本
**流程路径.<br>**
**WES/Panel: /data/wangty/JXH_test_tmp/work_test.<br>**
**WGS:  /data/wangty/JXH_test_tmp/work_test_WGS**
```


python3 extract_the_sample_list.py -h
 
usage: extract_the_sample_list.py [-h] --path PATH

Extract the sample list from the raw data path

optional arguments:
  -h, --help            show this help message and exit
  --path PATH, -p PATH  The clean sequencing data path, abs path is
                        recommended
```

运行命令：`python3 extract_the_sample_list.py --path {abs path of squence data}`

生成分析样本的Json文件
## 查看json 是否成功生成
```
less -S All_sample.json

EX:
	{
    "2210B001-F_test": {
        "R1": [
            "/data/usrname/JXH_test_tmp/test_data/2210B001-F_test/2210B001-F_test.R1.fq.gz"
        ],
        "R2": [
            "/data/usrname/JXH_test_tmp/test_data/2210B001-F_test/2210B001-F_test.R2.fq.gz"
        ]
    },
    "2210B001-M_test": {
        "R1": [
            "/data/usrname/JXH_test_tmp/test_data/2210B001-M_test/2210B001-M_test.R1.fq.gz"
        ],
        "R2": [
            "/data/usrname/JXH_test_tmp/test_data/2210B001-M_test/2210B001-M_test.R2.fq.gz"
        ]
    },
    "2210B001_test": {
        "R1": [
            "/data/usrname/JXH_test_tmp/test_data/2210B001_test/2210B001_test.R1.fq.gz"
        ],
        "R2": [
            "/data/usrname/JXH_test_tmp/test_data/2210B001_test/2210B001_test.R2.fq.gz"
        ]
    }

```



## 可调参数 TODO
除运行所用核心数外，本流程所有参数集中在`config.yml`中设置.<br>**使用前需要设置工作目录** .<br>**在课题组服务器上直接使用的时候，可以直接使用/data/wangty/JXH_test_tmp/work_test/bin or database.**

Example of config.yaml:
```
# config file
#
# # The requried files
software_scripts_path: '/data/usrname/JXH_test_tmp/work_test/bin'
data_base_path: '/data/usrname/JXH_test_tmp/work_test/data_base'
#
# # sample file
sample_json: 'All_sample.json'
#
# reference genome path
reference_panel_path: "/data/usrname/my_ref/hg38.fa"

# work directory
outdir: '/data/usrname/JXH_test_tmp/work_test'

# genome name
reference_panel_name: "hg38"

# basic argument for the snakemake

# rule ExtremeVar
Psoft: "ReVe,gt,0.7:VEST3,gt,0.6:REVEL,gt,0.4:CADD,gt,20:GERP++,gt,2"
MPsoft: 0.6
MAFS: "gnomAD_exome_ALL,0.001:gnomad312_AF,0.001:gnomAD_exome_EAS,0.001:gnomad312_AF_eas,0.001:ExAC_EAS,0.001:ExAC_ALL,0.001:esp6500siv2_all,0.001:1000g2015aug_eas,0.001:1000g2015aug_all,0.001"

# rule ExtremeVar2
MPsoft2: 0.1

# genebase
category: "genebased-hgmd-gene4denovo"

# panel_type
panel_type: "ASD"

# Juge
panel: True # 如果使用panel 设置为True，WES则为False
panel_gene: "gene.bed" # 设置Panel后需要提供gene bed文件

```

gene bed文件格式参考.<br>(panel 基因的基因组坐标，第一列染色体，第二列起始位置，第三列终止位置).<br>各个Panel基因（可根据需求设置某基因全部外显子或者直接给一个完整基因坐标）的基因组坐标
```
Ex:
chr1 1 100
chr1 235 599
....
```
## Gene bed TODO
```
后期可以建立一个基因与基因组坐标对应的库，后续只需要提供基因名称，记得得到它的全部外显子坐标或者完整坐标
```
安装完相关环境后：
```
/PATH/TO/SANKEMAKE_ENV/snakemake --cores 48 --use-conda
```
可根据实际可使用的核心数修改`--cores 48`
### 可选模式
WES/Panel/WGE



设置方式：
```

```

至生成**Joint GVCF**前，需要关心的参数是：
- Read Group信息
- [--interval-padding](https://gatk.broadinstitute.org/hc/en-us/articles/13832687299739-HaplotypeCaller#--interval-padding)



# 依赖与环境构建
本流程在Linux、MacOS环境下测试运行通过，理论上可以支持Windows

**注意**：参考文件可能需要较大的磁盘空间

## 必要依赖项 (需要先行安装)
- [Snakemake](https://snakemake.readthedocs.io/en/stable/getting_started/installation.html)
- conda
- [mamba](https://github.com/mamba-org/mamba) (optional, conda更快更好用的替代品)
- [GATK](https://github.com/broadinstitute/gatk/releases) (Genome Analysis Toolkit) 
- perl (后续部分分析使用perl脚本)
- JAVA (used by GATK, I use __java 20.0.2__ 2023-07-18)

### Software managed by Snakemake-conda
以下软件会在第一次运行Snakemake流程时自动安装。
⚠️⚠️⚠️ _**HPC用户注意！**_ ⚠️⚠️⚠️作业节点通常不能访问外网，请在登录节点使用如下命令预购建conda环境
```
snakemake --cores 1 --use-conda --conda-create-envs-only 
```
- fastqc
- fastp
- bwa
- samtools
## 其他参考文件
大部分的文件可以在此处下载
> [GATK Resource bundle](https://gatk.broadinstitute.org/hc/en-us/articles/360035890811-Resource-bundle) \
[Additional databases of annovar](https://annovar.openbioinformatics.org/en/latest/user-guide/download/)

请查看config.yaml以补充必要的文件

# 流程分步简介
## QC
### 修剪短读序列、接头（optional）
该步骤使用**fastp**在默认设置下运行

[官方文档](https://github.com/OpenGene/fastp)

由于大部分公司提供已经删除了测序接头的Clean Data，该步骤可能不是必须的，故设为可选项
> 1、fastp可以实现处理数据的一次性处理，包括过滤低质量，过滤adapter，截取reads，split分割大文件等操作 \
2、支持长reads，也就是不仅仅适用与illumina测序平台，还可以处理Pacbio和Ion torrent的测序数据 \  
3、直接输出质控和统计报告，包括json格式和html格式； \
4、使用c++写的，执行效率非常高；

### 生成可视化质控报告
该步骤使用**fastqc**在默认设置下运行

[官方文档](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)

该步骤生成一份html格式的报告，不会对数据产生更改

您可以在本地命令行（Unix/Linux）键入`fastqc`以启动本地窗口运行（如图所示）

<img width="792" alt="image" src="https://github.com/heihaheihaha/Call_variants_V1.0.0/assets/96468382/21bac703-e1d8-4037-ae21-45d5ce3e45e4">

其命令规范是：(执行`fastqc -h`以查看详细说明)
```
fastqc [-o output dir] [--(no)extract] [-f fastq|bam|sam] 
           [-c contaminant file] seqfile1 .. seqfileN
```

该报告包含以下内容：
- Basic statistics(基本信息)
- Per base sequence quality(序列测序质量统计)
<img width="689" alt="image" src="https://github.com/heihaheihaha/Call_variants_V1.0.0/assets/96468382/c62d7917-ff9c-4c9c-8689-8437db11cc2a">

- Per tail sequence quality(每个tail测序的情况)
- Per sequence quality scores
<img width="521" alt="image" src="https://github.com/heihaheihaha/Call_variants_V1.0.0/assets/96468382/4f341a9c-7673-41d0-9ec7-0858cdb39791">

- Per base sequence content(GC 含量统计)
<img width="679" alt="image" src="https://github.com/heihaheihaha/Call_variants_V1.0.0/assets/96468382/987003c5-d896-4c75-8675-2df923fca711">

- Per sequence GC content(序列平均GC含量分布图)
<img width="614" alt="image" src="https://github.com/heihaheihaha/Call_variants_V1.0.0/assets/96468382/c4177008-cba9-46b2-b3db-8c0580ad0b20">

- Sequnence lenth distribution(序列测序长度分布统计)
<img width="670" alt="image" src="https://github.com/heihaheihaha/Call_variants_V1.0.0/assets/96468382/d0685c1c-e70f-4389-903d-0ce36c678b3e">

- Adapter content(接头)
- Kmer content(重复短序列)

### 组装短读序列
该步骤使用**bwa（Burrows-Wheeler Aligner）**的bwa mem算法，组装短读序列并添加[Read Group](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups)信息

> 该步骤实际上不属于QC的范畴

bwa [文档](https://bio-bwa.sourceforge.net/)
> BWA 是一个软件包，BWA是一款将DNA序列mapping到参考基因组上的软件，例如比对到人类基因组。其由三个算法组成BWA-backtrack，BWA-SW和BWA-MEM组成。第一个算法设计用于读取约小于 100bp 的 Illumina 序列，而其余两种算法则用于读取 70bp 至 1Mbp 的较长序列。BWA-MEM 和 BWA-SW 具有相似的功能，例如长读支持和分割对齐，但 BWA-MEM 是最新的，通常建议用于高质量组装，因为它更快、更准确。对于 70-100bp Illumina 读取，BWA-MEM 比 BWA-backtrack 具有更好的性能。

> 受后续分析影响，目前仅测试了hg38上的组装，计划在下个版本中加入hg19

### 排序sam文件
按坐标对组装后的序列进行排列，输出bam文件（在GATK Best practices里可称为`uBAM`）

使用[`samtools sort`](https://www.htslib.org/doc/samtools-sort.html)

注意：我们计划将`fastp`, `bwa`, `samtool`使用[pipeline](https://en.wikipedia.org/wiki/Pipeline_(Unix))连接以实现性能最佳的并行, \
推荐分配的线程数为2-32-12，您可以自行设计`snakemake rule`以实现这一点。此外，在内存不充裕时，该步骤会有较高的I/O，请确保有足够的磁盘空间和读写速度。

### 标记重复序列（MarkDuplicates）
使用[Picard MarkDuplicates](https://broadinstitute.github.io/picard/picard-metric-definitions.html#DuplicationMetrics)

该步骤识别并标记识别重复的Reads，在本流程中，我们使用的是`MarkDuplicatesSpark`，不需要向其指定线程数，Spark会自行分配并行利用可用计算资源（无需在本地或是集群上安装Spark）,MarkDuplicatesSpark会自动为bam文件创建索引，为减少报错和便于更改和维护，建议在生成`bam`文件的rule添加一个检查并添加索引的`rule`

示例：
```
gatk MarkDuplicatesSpark \
    -I V300102848.bam \ # Input file
    -O V300102848.markdup.bam # Output file
```

> 重复reads源自 DNA 文库中相同的原始物理片段。重复有两种类型：PCR 重复和测序（各种光学混淆）重复。为了避免测量偏差并减少误差，应仅保留一份最佳质量的read。 \
MarkDuplicates 还会生成一个指标文件，指示单端和双端读取的重复次数。

是否需要标记重复序列**_众说纷纭_**，不在此处讨论。

值得注意是，该步骤会产生巨量的I/O——通常是数百GB数量级，对于SSD来说有着更快的速度和更短的寿命……请保证足够的内存或是磁盘空间以进行该步骤。

> See also: \
[Illumina DRAGEN DNA Pipeline](https://support.illumina.com/content/dam/illumina-support/help/Illumina_DRAGEN_Bio_IT_Platform_v3_7_1000000141465/Content/SW/Informatics/Dragen/GPipelineIntro_fDG.htm) / Duplicate Marking[Duplicate Marking](https://support.illumina.com/content/dam/illumina-support/help/Illumina_DRAGEN_Bio_IT_Platform_v3_7_1000000141465/Content/SW/Informatics/Dragen/DuplicateMarking_fDG.htm#:~:text=Marking%20or%20removing%20duplicate%20aligned,and%20lead%20to%20incorrect%20results.) 


### 碱基质量分数矫正(BQSR)
GATK [Base Quality Score Recalibration](https://gatk.broadinstitute.org/hc/en-us/articles/360035890531-Base-Quality-Score-Recalibration-BQSR-)

> BQSR 代表碱基质量分数重新校准。简而言之，它是一个数据预处理步骤，用于检测测序机在估计每个碱基调用的准确性时所产生的系统错误。 \
机器产生的碱基质量分数会受到各种系统性（非随机）误差的影响，导致数据中的碱基质量分数被高估或低估。其中一些错误是由于测序反应造成的，有些可能是由于设备的制造缺陷造成的。 \
碱基质量分数矫正（BQSR）是应用机器学习对这些错误进行经验建模并相应调整质量分数的过程。

该步骤使用三个rule，调用三个GATK tools，分别是：
- BaseRecalibrator
- ApplyBQSR
- AnalyzeCovariates (Optional)

其中，BaseRecalibrator 构建模型，ApplyBQSR 调整分数，AnalyzeCovariates绘制调整前后的对比图像

流程中，AnalyzeCovariates是可选的，并且被设计为对比2次矫正后的结果。该步骤通常不是必须的，并会带来一定的计算开销。

#### BaseRecalibrator
BaseRecalibrator要求如下参数
```
Required Arguments:
--input,-I <GATKPath>         BAM/SAM/CRAM file containing reads  This argument must be specified at least once.Required. 
--known-sites <FeatureInput>  One or more databases of known polymorphic sites used to exclude regions around known polymorphisms from analysis.  This argument must be specified at least once. Required. 
--output,-O <GATKPath>        The output recalibration table file to create  Required. 
--reference,-R <GATKPath>     Reference sequence file  Required. 
```
示例
```
# 注意BaseRecalibrator的输出是 .table 的文本文件
gatk BaseRecalibrator \
    -I V300102848.markdup.bam \
    -O V300102848.recal_data.table \
    -R /Volumes/ZHITAI/GATK_data/refs/hg38.fa \
    --known-sites /Volumes/ZHITAI/GATK_data/refs/Homo_sapiens_assembly38.dbsnp138.vcf.gz
```
#### ApplyBQSR
```
Required Arguments:

--bqsr-recal-file,-bqsr <File>Input recalibration table for BQSR  Required. 

--input,-I <GATKPath>         BAM/SAM/CRAM file containing reads  This argument must be specified at least once.
                              Required. 

--output,-O <GATKPath>        Write output to this file  Required. 
```



至此，bam文件已经完成质控


## Variants Calling（变异调用）
> 对于多样本，该步骤为Joint Calling

1. HaplotypeCaller \
    用于调用每个样本的变异并以 GVCF 格式保存调用。

2.GenomicsDBImport \
    将队列 GVCF 数据合并到 GenomicsDB 格式文件中。

3.GenotypeGVCFs \
    从合并的 GVCF 或 GenomicsDB 数据库中识别候选变异

### HaplotypeCaller
Required Arguments:
```
--input,-I <GATKPath>         BAM/SAM/CRAM file containing reads  This argument must be specified at least once.
                              Required. 
--output,-O <GATKPath>        File to which variants should be written  Required. 
--reference,-R <GATKPath>     Reference sequence file  Required. 
```

#### 关于`--emit-ref-confidence,-ERC`参数的设置
```
--emit-ref-confidence,-ERC <ReferenceConfidenceMode>
                              Mode for emitting reference confidence scores (For Mutect2, this is a BETA feature) 
                              Default value: NONE. Possible values: {NONE, BP_RESOLUTION, GVCF} 
```


The `--emit-ref-confidence` (or `-ERC`) option in GATK's HaplotypeCaller controls the emission of reference confidence scores, which are estimates of the likelihood that a genomic position is homozygous for the reference allele. The possible values for this option are `NONE`, `BP_RESOLUTION`, and `GVCF`.

1. **NONE**: This is the default setting, where HaplotypeCaller does not emit reference confidence scores. It's used when the focus is primarily on detecting variant sites. 主要用于检测变异位点。

2. **BP_RESOLUTION (Base Pair Resolution)**: This mode emits a confidence score for each base pair, providing detailed information about the reference confidence at every position in the genome. It's more data-intensive and used for analyses where detailed reference confidence is important. 用于参考置信度很重要的分析。

3. **GVCF (Genomic VCF)**: In this mode, HaplotypeCaller emits reference confidence scores in a genomic VCF format. It's less granular than `BP_RESOLUTION` but more data-efficient. In `GVCF` mode, HaplotypeCaller generates gVCF files containing both variant and non-variant sites, along with their confidence scores. This mode is used in workflows involving joint genotyping across multiple samples. 类似BP_RESOLUTION，但_**适用于Joint calling**_，故在多样本流程中使用该模式。

When running in either `GVCF` or `BP_RESOLUTION` modes, the confidence threshold is automatically set to 0, and this setting cannot be overridden from the command line. However, the threshold can be manually adjusted in the next step of the workflow (GenotypeGVCFs).

In summary, the `--emit-ref-confidence` option in HaplotypeCaller is crucial for determining how the tool reports confidence scores for regions believed to be homozygous for the reference allele, with the choice of mode depending on the specific requirements of the genomic analysis.
> **_See also_**: \
[HaplotypeCaller](https://gatk.broadinstitute.org/hc/en-us/articles/360037225632-HaplotypeCaller) \
[GATK](https://gatk.broadinstitute.org/hc/en-us)  [Technical Documentation](https://gatk.broadinstitute.org/hc/en-us/categories/360002310591-Technical-Documentation)  [Algorithms](https://gatk.broadinstitute.org/hc/en-us/sections/360007226771-Algorithms)[HaplotypeCaller Reference Confidence Model (GVCF mode) ](https://gatk.broadinstitute.org/hc/en-us/articles/360035531532-HaplotypeCaller-Reference-Confidence-Model-GVCF-mode-) 

#### 关于`--interval-padding,-ip` 参数的设置
```
--interval-padding,-ip <Integer>
                              Amount of padding (in bp) to add to each interval you are including.  Default value: 0.
```
具体设置可能因实际情况而有所差异

`--interval-padding`简单来说，即在`bed`文件提供的区域的两翼扩充多少bp,本流程中的默认值为**50**。
>  it's common to see a modest amount of padding, such as 100 bp or 200 bp, used in WES analyses

> _**See also**_: \
GATK论坛关于使用不同外显子组捕获试剂盒的数据间隔的[讨论](https://gatk.broadinstitute.org/hc/en-us/community/posts/360056487811-What-intervals-to-use-in-HaplotypeCaller-GenotypeGVCF-for-exome-data-sequenced-with-different-capture-kits) \
[GATK Documentation on Intervals and Interval Lists](https://gatk.broadinstitute.org/hc/en-us/articles/360035531852-Intervals-and-interval-lists) \
[Algorithm and HaplotypeCaller Documentation](https://gatk.broadinstitute.org/hc/en-us/sections/360007226612) 

示例
```bash
gatk HaplotypeCaller \
    -I V300102848.bqsr.markdup.bam \
    -O V300102848.ht.vcf.gz \
    -R /Volumes/ZHITAI/GATK_data/refs/hg38.fa \
    -L /Users/heihaheihaha/data_a/tl_WES/NanoWESv2-hg38.bed \
    -D /Volumes/ZHITAI/GATK_data/refs/Homo_sapiens_assembly38.dbsnp138.vcf.gz \
    -ip 100 \
    -ERC GVCF
```
对于单样本/panel分析，可设置`-ERC BP_RESOLUTION`

#### GenotypeGVCFs
**“HaplotypeCaller”**用于识别每个样本中的变体（SNPs和indel),**GenotypeGVCFs**从多个样本中提取这些单独的GVCF文件，并进行联合基因分型。



## 变异过滤 
可以通过设置`rule end`使流程在变异调用后即结束
为保证流程的可解释性和稳定，此处采用硬过滤（Hard filtering）的方法。对于更多样本的分析，VQSR可能是更好的办法。

GATK 关于设置Hard filtering参数的[建议](https://gatk.broadinstitute.org/hc/en-us/articles/360035890471-Hard-filtering-germline-short-variants)

流程中参数默认设置如下：

For SNP：
```
--filter-expression 'QUAL<30.0' --filter-name 'LOW_QUAL' \\
--filter-expression 'QD<2.0' --filter-name 'LOW_QD' \\
--filter-expression 'FS>60.0' --filter-name 'HIGH_FS' \\
--filter-expression 'MQ<40.0' --filter-name 'LOW_MQ' \\
--filter-expression 'MQRankSum<-12.5' --filter-name 'LOW_MQRS' \\
--filter-expression 'ReadPosRankSum<-8.0' --filter-name 'LOW_RPRS' \\
--filter-expression 'SOR>3.0' --filter-name 'HIGH_SOR' \\
```
For indel：
```
--filter-expression 'QUAL<30.0' --filter-name 'LOW_QUAL' \\
--filter-expression 'QD<2.0' --filter-name 'LOW_QD' \\
--filter-expression 'FS>200.0' --filter-name 'HIGH_FS' \\
--filter-expression 'ReadPosRankSum<-20.0' --filter-name 'LOW_RPRS' \\
--filter-expression 'SOR>10.0' --filter-name 'HIGH_SOR' \\
```
## FilterVar TODO

## ExtremeVar.pl
该步骤主要使用annovar对vcf文件进行注释,见database.list以查看具体的使用的数据库。

#LOG
流程会在`Snakefile`同级文件夹下创建一个`log`文件夹下
# 致谢
本流程使用以下开源软件
- fastqc
- fastp
- bwa
- samtools
- [annovar](https://annovar.openbioinformatics.org/en/latest/)
- Snakemake
- conda
- mamba
- GATK (Genome Analysis Toolkit) 
- perl 
- JAVA 
- [bamdst](https://github.com/shiquan/bamdst)
