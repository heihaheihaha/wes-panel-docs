---
layout: post
title:  "Problems and discussion"
date:   2023-12-18 15:02:41 +0800
categories: discussion
---
# 1 Warnings
1.	低质量输入
a)	空输入、路径错误	加入check
b)	质控差
检查深度和fastp输出，
 
2.	空输出检查、break/warning
讨论：这个问题应该集中在后续perl脚本处理的任务中（含DenovoCNN），最好在各自的脚本里检查输出文件并提bash error

> 后续perl有关流程由mxx师兄标准化，仅关注Snakemake流程中的error

# 2 MarkduplicatesSpark
> 需要Mark duplicates，加入该步骤，取消samtools sort
 
# 3 BaseRecalibrator’s known site

常用：更改为下述参数
dbsnp_146.hg38.vcf.gz
Homo_sapiens_assembly38.known_indels.vcf.gz
Mills_and_1000G_gold_standard.indels.hg38.vcf.gz
https://gatk.broadinstitute.org/hc/en-us/articles/360035890811-Resource-bundle

# 4 Trio ID触发家系 DenovoCNN，共分离
是否考虑非常规家系的共分离？

> 津臣老师：先处理raw data 到vcf，家系的分析有现成的脚本

# 5 多样本（含Trio）输入触发 Jointcalling

# 6 留痕
1.	Log snakemake
2.	snakemake –forcecall –n -h

# 7 简化操作流程
根据实际情况自定义脚本制作输入参数
