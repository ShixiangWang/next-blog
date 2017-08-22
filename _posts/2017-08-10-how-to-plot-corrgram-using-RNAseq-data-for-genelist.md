---
title: 怎么分析和展示RNAseq基因表达数据中基因的相关性
date: 2017-08-08
categories: 
- R
tags: 
- TCGA analysis
- R

---


## 介绍

TCGA是癌症基因组分析中相当流行的数据库，针对里面数据的挖掘结果、软件工具发表了许多CNS文章，不过现在已经被整合进[GDC数据平台](https://portal.gdc.cancer.gov/)了。虽然现在测序技术发展的很快，单个样本测序成本比较之前而言低了很多，但是对于单个课题组研究而言，对大量样本的测序成本还是有些难以承受。TCGA的数据集提供了一个很好的平台，我们既可以分析它衍生新的课题，也可以通过它为自己分析的结果佐证。今天的分析用的就是TCGA肺腺癌的数据集（TCGA-LUAD），可以点击[这里](https://xenabrowser.net/datapages/?cohort=TCGA%20Lung%20Adenocarcinoma%20(LUAD))进入UCSC的数据集资源库下载。

<!-- more -->

RNAseq的结果中包含了数万个基因的表达值，而我们往往感兴趣的只是少数。基于一些先验知识，我们可能想要查看某些基因之间的相关性如何，以辅助构想这些基因之间的关系模式是怎样的。一种非常直观的办法是对基因两两建立回归模型（线性回归或者广义线性回归）。这样需要画的图和构建的模型根据你想要查看基因数的变化会有很多变化，虽然可以通过循环之类的方式实现，但我并不推荐。懒人表示喜欢简单易懂的，有一种非常简约的办法：构造基因表达的相关系数矩阵，然后展示它。



## R实现

下面看怎么用`corrgram`包实现：

首先构建两个用来读写`tsv`文件（table键分隔的文件，TCGA数据集以这种格式存储）的函数。

```R
read.tsv <- function(file=file, header=TRUE, sep="\t"){ 
    return(read.table(file=file, header=header, sep=sep))}

write.tsv <- function(x,file, quote=FALSE, sep="\t", row.names=FALSE, col.names=TRUE){
    return(write.table(x=x, file=file, quote=quote, sep=sep,
                       row.names = row.names, col.names=col.names))
}
```

非常简单，就是封装了一下R本身自带的`write.table`与`read.table`函数。因为我并没有看到R自带`tsv`文件处理的函数，所以封装了这两个函数，用起来更方便。

构建一个函数来实现展示基因表达量相关性的功能，它主要完成3件事情，根据输入参数提取出进行分析的数据集，将这个数据集作为参数传入`corrgram`函数，然后将生成的图形输出。`corrgram()`函数自动会对传入的数据集变量进行相关分析，然后生成图形，所以我们没必要在此之前用`cor`函数处理。

需要传入函数的参数有6个，必要的有5个。所有参数的含义我已经用英文进行了注释说明（我的Rstudio用不了中文）。

简单解释一下，`gene.list`是想要查看的基因集（最后只会找出RNAseq基因ID中能找到的）；`project_code`是项目代码，比如`TCGA-LUAD`，这里也可以当job id设置，用以区分；`project.clinical`与`project.exp`就是临床信息数据集和RNA表达数据集（如果引用非TCGA数据集时变量名对不上，自己改下哈）；`outdir`设置输出文件目录；`ID_transform`用来控制样本ID的转换，TCGA临床数据用的是`-`分隔样本ID，而RNAseq结果中用的是`.`分隔，所以需要转换。如果参考使用下面函数时有什么问题，争取自己动手改改，也可以文章下方留言。

因为RNAseq数据中包含的病人类型不一，所以在分析所有样本后，我增加提取癌症病人的代码，主要是原位瘤和转移瘤。前者在我见过的TCGA数据集肯定有，后面则不一定，所以用`if`语句控制了下分析流程。

```R
gene_exp.corr <- function(gene.list, project_code, project.clinical, project.exp, outdir, ID_transform=TRUE){
    # Arguments:
    # gene.list: a list of gene you want to analyze their expression correlation
    # project_code: data project name or name you want to specify this analysis
    # project.clinical: clinical information about samples, data.frame format
    # project.exp: normalized gene expression (RNA seq) about samples, data.frame format
    # ID_transform: sometimes clinical information use "-" as separate symbol for sample ID,
    #               we need it to be the same as it in project.exp data
    # one sample ID example: in clinical information, one sample may be marked by "TCGA-3N-A9WB-06",
    #                        in RNA seq data.set, this sample is "TCGA.3N.A9WB.06". If it is not, set ID_transform=FALSE. 
    
    # note: you need to install "corrgram" package before use this function
    
    gene_exp.list <- subset(project.exp, sample%in%gene.list)
    rownames(gene_exp.list) <- gene_exp.list[,1]
    gene_exp.list <- gene_exp.list[,-1]
    gene_exp.list <- t(gene_exp.list)
  
    library(corrgram)
  
    if(ID_transform){
        project.clinical$sampleID = gsub("-",".",project.clinical$sampleID, fixed = TRUE)
    }
    n.gene <- ncol(gene_exp.list)
    # all samples
    pdf(paste(outdir,project_code,"_all_sample_genelist_expression_corrgram.pdf", sep=""))
    corrgram(gene_exp.list, lower.panel = panel.shade, order = TRUE,
             upper.panel = panel.pie, text.panel = panel.txt,
             main=paste("Corrgram of ", n.gene," Genes Expression in ", project_code, sep = ""))
    dev.off()
    
    # choose tumor sample
    # table(project.clinical$sample_type)
    primary_tumor <- "Primary Tumor"
    Metast_tumor  <- "Metastatic"
    primary_tumor.id <- project.clinical[project.clinical$sample_type==primary_tumor,]$sampleID
    Metast_tumor.id  <- project.clinical[project.clinical$sample_type==Metast_tumor,]$sampleID
    
    if(length(primary_tumor.id)<2 & length(Metast_tumor.id<2)){
        stop("Maybe your data have something wrong. Please check it!")
    }else{
        if(length(primary_tumor.id)<2){
            stop("I don't think it's reasonable that there are less than 2 primary tumor samples.")}
        
        gene_exp.list.primary <- subset(gene_exp.list, rownames(gene_exp.list)%in%primary_tumor.id)
        pdf(paste(outdir,project_code,"_primary_tumor_sample_genelist_expression_corrgram.pdf", sep=""))
        corrgram(gene_exp.list.primary, lower.panel = panel.shade, order = TRUE,
                 upper.panel = panel.pie, text.panel = panel.txt,
                 main=paste("Corrgram of ", n.gene," Genes Expression in ", project_code, sep = ""))
        dev.off()
        
        if(length(Metast_tumor.id)<2){
            cat("It seems has no Metastatic sample in this analysis. \n")
            return(0)
        }
        gene_exp.list.Metast  <- subset(gene_exp.list, rownames(gene_exp.list)%in%Metast_tumor.id)
        pdf(paste(outdir,project_code,"_Metastatic_sample_genelist_expression_corrgram.pdf", sep=""))
        corrgram(gene_exp.list.Metast, lower.panel = panel.shade, order = TRUE,
                 upper.panel = panel.pie, text.panel = panel.txt,
                 main=paste("Corrgram of ", n.gene," Genes Expression in ", project_code, sep = ""))
        dev.off()}
}

```

下面拿数据实测一下。

```R
# analysis
setwd("~/your_work_directory/")
options(stringsAsFactors = FALSE)

# 参数设置
outdir <- "../results/"        # 设置输出目录，找个能找着的就行
project_code <- "TCGA-LUAD"
gene_list <- c("ERF", "ETV3", "ELF3", "ELF5", "ESE3", "ELK1", "ELK4",
               "ELK3", "ETV6", "ETV7")
project.clinical <- read.tsv("~/LiuLab/apobec3b/dataset/LUAD_clinicalMatrix")
project.exp <- read.tsv("~/LiuLab/apobec3b/dataset/LUAD_HiSeqV2_PANCAN")

# 调用函数
gene_exp.corr(gene.list = gene_list, project_code = project_code, project.clinical = project.clinical, project.exp = project.exp, outdir)
```

结果以`pdf`格式保存，毕竟矢量图质量好。

还会返回：

```R
It seems has no Metastatic sample in this analysis. 
[1] 0
```

因为这个数据集没有转移瘤病人。

看看输出的图形结果吧，这里只放一张原位癌病人的图当做demo。

![](/images/demo_corrgram.png)

关于图形的输出效果可以参考`corrgram`包参数（help一下）设定，《R实战》书中有它的介绍。这里设定的是下三角用阴影图，上三角用饼图，两种结果的解释是一致的。

```R
corrgram(gene_exp.list.primary, lower.panel = panel.shade, order = TRUE,
                 upper.panel = panel.pie, text.panel = panel.txt,
                 main=paste("Corrgram of ", n.gene," Genes Expression in ", project_code, sep = ""))
```

左下角的方块图和右上角的饼图显示的结果完全相同（展示的是变量（基因）相关矩阵）：

- 蓝色和从左下指向右上的斜杠表示两个变量正相关。反过来，红色和从左上指向右下的斜杠表示呈现负相关。色彩越深，饱和度越高，说明变量相关性越大。
- 右上角的饼图展示同样信息。颜色功能同上，相关性大小是由被填充的饼图块的大小来展示。顺时针填充为正相关，逆时针填充为负相关。
- `corrgram`包的`corrgram`函数设置了`order=TRUE`，相关矩阵会使用主成分分析方法对变量重排，有点聚类的效果，展示了变量的相关关系模式。


--------------------------------

**参考**：《R实战》第二版
