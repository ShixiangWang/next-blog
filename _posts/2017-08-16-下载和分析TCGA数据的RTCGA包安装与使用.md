---
title: 下载和分析TCGA数据的RTCGA包安装与使用
date: 2017-08-16
categories: R
tags:
- Bioconductor
- R
- TCGA
- RTCGA
---

本文来自对官方文档的翻译学习，原3-29发于[wordpress](https://moiedotblog.wordpress.com/2017/03/29/introduction-of-rtcga-package-in-r/)。因为网络不太好，那个博客已差不多弃用了。

## 安装

在R或者Rstudio交互界面输入：

```R
source("http://bioconductor.org/biocLite.R")
 biocLite("RTCGAToolbox")
```

Windows下，如果安装出现问题，请查看依赖包是否完整。我安装时发现XML包可能需要单独安装。

如果你是Linux系统，而且XML包一直安装不上，请仔细查看错误信息。有可能是你的系统没有XML和curl配置，导致不能安装XML以及Rcurl包（具体依据错误信息分析）。在终端下输入

```shell
sudo apt-get install libxml2-dev
 sudo apt-get install libcurl4-gnutls-dev
```

完成后接着安装XML包和RCurl包；安装RTCGA工具包。

## 数据的查看与导入

首先导入工具包：

```
library(RTCGAToolbox)
```

查看合法的数据集别名：


```
# Valid aliases
 > getFirehoseDatasets()
```

查看合法标准数据的运行日期：

查看合法的分析运行日期：

```
# Valid analysis running dates (will return 3 recent date)
 > gisticDate = getFirehoseAnalyzeDates(last=3)
 > gisticDate
 [1] "20160128" "20150821" "20150402"
```

日期和数据集确定了你通过getFirehoseData函数需要获取的数据。
```
# READ mutation data and clinical data
 brcaData = getFirehoseData (dataset="READ", runDate="20150402",forceDownload = TRUE,
 Clinic=TRUE, Mutation=TRUE)
```
`getFirehoseData`函数需要设置的一些参数：
```
- dataset: Users should set cohort code for the dataset they would like to download. List can be accessiable via getFirehoseDatasets() like as explained above.
- runDate: Firehose project provides different data point for cohorts. Users can list dates by using function above. “getFirehoseRunningDates()”
- gistic2_Date: Just like cohorts Firehose project runs their analysis pipelines to process copy number data with GISTIC2 (Mermel, C. H. and Schumacher, S. E. and Hill, B. and Meyerson, M. L. and Beroukhim, R. and Getz, G 2011). Users who want to get GISTIC2 processed copy number data should set this date. List can be accessible via “getFirehoseAnalyzeDates()”
```

下面是一些提供不同数据类型的逻辑值：

```
- RNAseq_Gene
- Clinic
- miRNASeq_Gene
- RNAseq2_Gene_Norm
- CNA_SNP
- CNV_SNP
- CNA_Seq
- CNA_CGH
- Methylation
- Mutation
- mRNA_Array
- miRNA_Array
- RPPA
```

其他一些参数：
```
- forceDownload: 强制下载。
- fileSizeLimit: 默认为500MB，可以自己根据这个参数设置。
- getUUIDs: Firehose为每个样本都提供了一个叫做UUID的TCGA barcodes编码，可以通过提供这个参数获取。
```

## 分析功能

RTCGAToolbox提供了显示数据基本信息的功能函数。分析功能包括差异基因表达分析、CN与GE相关分析、突变频率表和报告表等。

“玩具”数据集

一个展示基本数据集信息结构的无意义数据集。

## 差异基因表达
```
# Differential gene expression analysis for gene level RNA data.
 diffGeneExprs = getDiffExpressedGenes(dataObject=RTCGASample,DrawPlots=TRUE,
 adj.method="BH",adj.pval=0.05,raw.pval=0.05,
 logFC=2,hmTopUpN=10,hmTopDownN=10)
```

RTCGA工具集使用了voom包和limma包的函数做这个功能分析。每个经过TCGA项目处理过的样本都有一个特定的包含组织源信息的barcode数。RTCGA工具集利用这个barcode信息将每个样本分成正常和肿瘤组以进行差异基因表达分析。因为voom需要RNASeq data的原始计数，所以标准化的数据是不能用来做这个分析。

该函数会返回一个列表，其中每个成员都是一个"DGEResult"对象。该对象有一个top table，包含基因log2倍数表达量变化及其显著性地矫正p值，函数默认会用初始p值、矫正p值以及log倍数改变过滤结果。我们可以通过adj.pval，raw.pval，logFC参数调整进行定制。函数采用Benjamini & Hochberg方法为p值矫正，更多信息可以通过?p.adjust查看。函数默认只会画出100个上调和下调基因地热图，我们可以使用hmTopUpN和hmTopDownN参数进行调整。
```
# Show head of expression outputs
> diffGeneExprs
 [[1]]
 Dataset:RNASeq
 DGEResult object, dim: 15 6
```
```
# Dataset: RNASeq
> showResults(diffGeneExprs[[1]])
 Dataset: RNASeq
 logFC AveExpr t P.Value adj.P.Val B
 TAP2 5.288573 1.743410 76.38279 7.496531e-76 4.760297e-73 150.82307
 GRTP1 6.187648 2.193494 48.25410 2.030875e-60 6.448029e-58 123.44071
 ENPP5 7.215676 2.707012 45.79069 1.110692e-58 2.350964e-56 120.11226
 APH1B 8.118533 3.158063 37.31744 5.796089e-52 6.134194e-50 106.20724
 INSR 8.055541 3.126521 32.82711 7.967149e-48 1.945823e-46 97.36010
 MINPP1 7.097777 2.647495 30.36322 2.431258e-45 3.508747e-44 91.95723

toptableOut = showResults(diffGeneExprs[[1]])

If “DrawPlots” set as FALSE, running code above won’t provide any output figure.

Voom + limma: To voom (variance modeling at the observational level) is to estimate the mean-variance relationship robustly and non-parametrically from the data. Voom works with log-counts that are normalized for sequence depth, in particular with log-counts per million (log-cpm). The mean-variance is fitted to the gene-wise standard deviations of the log-cpm, as a function of the average log-count. This method incorporates the mean-variance trend into a precision weight for each individual normalized observation. The normalized log-counts and associated precision weights can then be entered into the limma analysis pipeline, or indeed into any statistical pipeline for microarray data that is precision weight aware(Smyth, G. K 2004; Law, C. W. and Chen, Y. and Shi, W. and Smyth, G. K 2014). Users can check the following publications for more information about methods:

limma : Smyth, G. K. (2004). Linear models and empirical Bayes methods for assessing differential expression in microarray experiments. Statistical Applications in Genetics and Molecular Biology, Vol. 3, No. 1, Article 3.

Voom: Law, CW, Chen, Y, Shi, W, Smyth, GK (2014). Voom: precision weights unlock linear model analysis tools for RNA-seq read counts. Genome Biology15, R29.
```

## 基因表达与拷贝数之间的相关性

getCNGECorrelation 函数返回拷贝数与基因表达数据之间的相关系数和矫正p值。This function takes main dataobject as an input (uses gene copy number estimates from GISTIC2 (Mermel, C. H. and Schumacher, S. E. and Hill, B. and Meyerson, M. L. and Beroukhim, R. and Getz, G 2011) algorithm and gen expression values from every platform (RNAseq and arrays) to prepare return lists. List object stores “CorResult” object that contains results for each comparison.)

```
> corrGECN = getCNGECorrelation(dataObject=RTCGASample,adj.method="BH",
 + adj.pval=0.9,raw.pval=0.05)
 > corrGECN
 [[1]]
 Dataset:RNASeq
 CorResult object, dim: 43 4
```
```
> showResults(corrGECN[[1]])
 Dataset: RNASeq
 GeneSymbol Cor adj.p.value p.value
 2 SEPHS1 -0.3769382 0.6287624 0.01650501
 9 MSMB -0.3404472 0.8472802 0.03158898
 39 PRMT3 -0.3806078 0.6287624 0.01540088
 85 HTR3A 0.3378158 0.8472802 0.03301479
 91 PHLDB1 -0.3260183 0.8688328 0.04007191
 101 EMP1 0.3814696 0.6287624 0.01515084
```
```
> corRes = showResults(corrGECN[[1]])
 Dataset: RNASeq
 GeneSymbol Cor adj.p.value p.value
 2 SEPHS1 -0.3769382 0.6287624 0.01650501
 9 MSMB -0.3404472 0.8472802 0.03158898
 39 PRMT3 -0.3806078 0.6287624 0.01540088
 85 HTR3A 0.3378158 0.8472802 0.03301479
 91 PHLDB1 -0.3260183 0.8688328 0.04007191
 101 EMP1 0.3814696 0.6287624 0.01515084
```
相关分析之后，RNASeq data(如果采用的是RNASeq)会被标准化。相关分析用Benjamini & Hochberg adjustment for p values。这里采用的是皮尔逊积矩相关系数去检测两个配对样本之间的关联。如果样本服从独立正态分布，统计检验服从t分布，自由度为length(x)-2。更多详细信息，使用?cor.test函数获取。

## 突变频率

getMutationRate函数计算得到一个关于每个基因突变频率的数据框。

```
# Mutation frequencies
> mutFrq = getMutationRate(dataObject=RTCGASample)
> head(mutFrq[order(mutFrq[,2],decreasing=TRUE),])
 Genes MutationRatio
 FCGBP FCGBP 0.46
 NF1 NF1 0.31
 ASTN1 ASTN1 0.24
 ODZ4 ODZ4 0.22
 BRWD1 BRWD1 0.22
 SYNE2 SYNE2 0.22
```

单因素（生存）分析

单因素生存分析被成为是一种能够为临床诊断提供价值信息的方法。该函数创建2个或者3个基于表达数据的群组。(If the dataset has RNASeq data, data will be normalized for survival analysis.)。 如果group设置为2，工具包将通过独立基因的表达均值创建两个群组；如果group设置为3，这些群组被定义为：第一分位数的样品（expression 3rd Q），以及两者之间。

单因素生存分析函数需要生存数据，这可以通过临床数据框获得。生存数据第一列是sample barcodes，第二列是time，最后一列是event data。下面说明怎样获取临床数据，生成生存数据，并进行单因素生存分析（Univariate survival analysis）。

```
# Creating survival data frame and running analysis for
# FCGBP which is one of the most frequently mutated gene in the toy data
# Running following code will provide following KM plot.
> clinicData head(clinicData)
 Composite.Element.REF yearstobirth vitalstatus daystodeath
 TEST.00.0026 value 53 0 NA
 TEST.00.0052 value 50 0 NA
 TEST.00.0088 value 56 0 NA
 TEST.00.0056 value 56 0 NA
 TEST.00.0023 value 56 0 NA
 TEST.00.0092 value 52 0 NA
 daystolastfollowup neoplasm.diseasestage pathology.T.stage
 TEST.00.0026 1183 stage iiia stage iiia
 TEST.00.0052 897 stage iib stage iib
 TEST.00.0088 1000 stage iib stage iib
 TEST.00.0056 1134 stage iia stage iia
 TEST.00.0023 794 stage iib stage iib
 TEST.00.0092 1104 stage iia stage iia
```
```
clinicData = clinicData[,3:5]
 clinicData[is.na(clinicData[,3]),3] = clinicData[is.na(clinicData[,3]),2]
 survData <- data.frame(Samples=rownames(clinicData),
 Time=as.numeric(clinicData[,3]),
 Censor=as.numeric(clinicData[,1]))
 getSurvival(dataObject=RTCGASample,geneSymbols=c("FCGBP"),sampleTimeCensor=survData)
```

## 数据导出

可以用getData()函数将下载数据从FirehoseData对象导出。
```
# Note: This function is provided for real dataset test since the toy dataset is small.
 RTCGASample

TEST FirehoseData object
 Available data types:
 Clinical: A data frame, dim: 100 7
 RNASeqGene: A matrix withraw read counts or normalized data, dim: 800 80
 GISTIC: A FirehoseGISTIC object to store copy number data
 Mutations: A data.frame, dim: 2685 30
 To export data, you may use getData() function.
```
```
RTCGASampleClinical = getData(RTCGASample,"Clinical")
 RTCGASampleRNAseqCounts = getData(RTCGASample,"RNASeqGene")
 RTCGASampleCN = getData(RTCGASample,"GISTIC")
```
## 重述原始文章中的BRCA结果

Following code block is provided to reproduce case study in the RTCGAToolbox paper(Samur MK. 2014).
```
# BRCA data with mRNA (Both array and RNASeq), GISTIC processed copy number data
# mutation data and clinical data
# (Depends on bandwidth this process may take long time)
 brcaData = getFirehoseData (dataset="BRCA", runDate="20140416", gistic2_Date="20140115",
 Clinic=TRUE, RNAseq_Gene=TRUE, mRNA_Array=TRUE, Mutation=TRUE)

# Differential gene expression analysis for gene level RNA data.
# Heatmaps are given below.
 diffGeneExprs = getDiffExpressedGenes(dataObject=brcaData,DrawPlots=TRUE,
 adj.method="BH",adj.pval=0.05,raw.pval=0.05,
 logFC=2,hmTopUpN=100,hmTopDownN=100)
# Show head for expression outputs
 diffGeneExprs
 showResults(diffGeneExprs[[1]])
 toptableOut = showResults(diffGeneExprs[[1]])
# Correlation between expresiion profiles and copy number data
 corrGECN = getCNGECorrelation(dataObject=brcaData,adj.method="BH",
 adj.pval=0.05,raw.pval=0.05)

corrGECN
 showResults(corrGECN[[1]])
 corRes = showResults(corrGECN[[1]])

# Gene mutation frequincies in BRCA dataset
 mutFrq = getMutationRate(dataObject=brcaData)
 head(mutFrq[order(mutFrq[,2],decreasing=TRUE),])

# PIK3CA which is one of the most frequently mutated gene in BRCA dataset
 # KM plot is given below.
 clinicData <- getData(brcaData,"Clinical")
 head(clinicData)
 clinicData = clinicData[,3:5]
 clinicData[is.na(clinicData[,3]),3] = clinicData[is.na(clinicData[,3]),2]
 survData <- data.frame(Samples=rownames(clinicData),
 Time=as.numeric(clinicData[,3]),
 Censor=as.numeric(clinicData[,1]))
 getSurvival(dataObject=brcaData,geneSymbols=c("PIK3CA"),sampleTimeCensor=survData)
```
## 报告图

这里的这个函数使用RCircos(Zhang, H. and Meltzer, P. and Davis, S 2013)为输入数据集提供了整体的环形图结果。输入需要[differential gene expression analysis results (max results for 2 different platforms), copy number data estimates from GISTIC2 (Mermel, C. H. and Schumacher, S. E. and Hill, B. and Meyerson, M. L. and Beroukhim, R. and Getz, G 2011) and mutation data.]。
```
# Creating dataset analysis summary figure with getReport.
# Figure will be saved as PDF file.
 library("Homo.sapiens")
 locations = genes(Homo.sapiens,columns="SYMBOL")
 locations = as.data.frame(locations)
 locations <- locations[,c(6,1,5,2:3)]
 locations <- locations[!is.na(locations[,1]),]
 rownames(locations) <- locations[,1]
 getReport(dataObject=brcaData,DGEResult1=diffGeneExprs[[1]],
 DGEResult2=diffGeneExprs[[2]],geneLocations=locations)
```
## 数据对象

RTCGASample数据虽然是“玩具”，但是也是FirehoseData数据对象，存储了RNAseq, copy number, muatation, clinical data for artificially created dataset。
```
data(RTCGASample)
 ## RTCGASample dataset is artificially created for function test.
 ## It isn't biologically meaninful and it has no relation with any cancer type.
 ## For real datasets, please use client function to get data from data portal.
```

## 参考官方文档:RTCGAToolbox