# Human Genome Reference  


## 參考序列的起源
1990年代，人類基因組計劃開始致力於定序全基因體，想定出的範圍包含：
> The complete human genome consists of 22 diploid chromosomes (1 − 22), two sex chromosomes (X and Y) and maternally inherited mitochondrial DNA (mtDNA). 

初期定序方法是利用Sanger sequencing 加 genomic shotgun 一小段定出來並組裝（assembly）耗時長且費力，直到2003年才宣布完成第一條全基因體序列資料，這份珍貴資訊也被當作參考序列應用到研究中；隨定序技術的進步，參考序列也不停經歷修正與重新定序，現在已有多種版本可選擇。

## 參考序列的版本差異

### GRCh37
* 發行時間：2009年
* 檔案名稱：GRCh37.p13.genome.fasta
* 版本說明：GRCh37 全名 Genome Reference Consortium Human Build 37，是由人類參考序列聯盟(GRC)整理定序的參考序列，定序資料一部分承襲Human Genome Project完成的，另一部分利用WGS shotgun定出unfilled gap 和 error sequence。
最終版本包含chr1-22,X,Y和MT(mitochondria DNA)，並依定序情況分成三大類：

> unlocalized sequences：知道来自哪條染色體但不知道具體位置的序列
unplaced sequences：知道来自人類基因组序列，但不知道與染色體的關係
alternate loci：来自基因组特定區域，代表該區域序列的多樣性

* 補充1：
GRCh37還有版本更新，NCBI紀錄
    * [GRCh37](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.13/#/st) (2009)
    * [GRCh37.p13](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.25/) (2013)

* 補充2:GRC是由以下學術機構組成
  * [The Wellcome Trust Sanger Institute](https://www.sanger.ac.uk/science/programmes/)
  * [The McDonnell Genome Institute at Washington University](https://www.genome.wustl.edu)
  * [The European Bioinformatics Institute](https://www.ebi.ac.uk)
  * [The National Center for Biotechnology Information](https://www.ncbi.nlm.nih.gov)

### hg19
* 發行時間：2009年
* 檔案名稱：ucsc.hg19.fasta
* 版本說明：是由UCSC Genome Browser基於GRCh37建立發行的版本，蠻多人會將兩版本通用，和GRCH37的差異如下：
  * 部分contig名稱更動
  * 染色體編號：GRCh37為 1, 2, 3, 4, ….X, Y, MT，hg19為 chr1, chr2, chr3, chr4, …..chrX, chrY, chrM
  * 粒線體版本不同：是兩版本最大的差異，GRCh37納入的是修正版 [NC_012920](https://www.genome.jp/dbget-bin/www_bget?refseq:NC_012920)，而hg19是使用舊版 [NC_001807](https://)

* 補充:
[UCSC官網](https://hgdownload.soe.ucsc.edu/downloads.html)可下載序列資料

### b37
* 檔案名稱：Homo_sapiens_assembly19.fasta
* 版本說明：是由Broad Institute基於GRCh37建立發行的版本，序列資料包含 GRCh37 使用GATK tool常會以此版作為參考序列。

### humanG1Kv37（hs37d5）

* 檔案名稱：
   * 不包含decoy: human_g1k_v37.fasta 
   * 包含decoy: hs37d5.fa
* 版本說明：是由 1000 genomes Project 所完成的版本，相當於b37版本，不同之處在於此版有包含或不包含 [decoy sequence](https://www.ncbi.nlm.nih.gov/nuccore/NC_007605) (human herpesvirus 4 type 1)兩種。
* 補充：目前研究多使用hs37d5作為參考序列，因在後續分析有最高的準確度，這說明了加上decoy seuence後能增加序列比對的數量。

### GRCh38

* 發行時間：2013年
* 版本說明：和GRCh37版本相比，因定序處理的技術提升，定出資訊更多，如 exon region, SNV and InDels等；GRCh38也試著做annotation of the centromere regions. 

![](https://i.imgur.com/sqfEIig.png)

而UCSC Genome Browser為了避免過往版本號碼不一致 (GRCh37 vs. hg19)造成混淆，變將其版本號碼跳動至hg38，和GRCh38可相通（但仍需座標轉換）。

* 補充：
GRCh38還有版本更新，NCBI紀錄
[GRCh38](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.26/) (2013)
[GRCh38.p13](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.39/) (2019)

### GRCh38版本比較


|    version    |           GATK           |              Illumina                | bwakit |              1000 Genome              |                         NCBI                         |
|:-------------:|:-----------------------------:|:------------------------------------------:|:-----------:|:------------------------------------------:|:---------------------------------------------------------:|
|     decoy     |               +               |                     +                      |      +      |                     +                      |                             +                             |
|      ALT      |               +               |                     +                      |      +      |                     +                      |                             +                             |
|      HLA      |               +               |                     +                      |      +      |                     +                      |                             -                             |
| Contig_Number |             3366              |                    3366                    |    5751     |                    3366                    |                           2841                            |
|   file name   | Homo_sapiens_assembly38.fasta | GRCh38_full_analysis_set_plus_decoy_hla.fa |  hs38DH.fa  | GRCh38_full_analysis_set_plus_decoy_hla.fa | GCA_000001405.15_GRCh38_full_plus_hs38d1_analysis_set.fna |

* 版本差異及使用心得：在GRCh38版本中，依照 bwa-kit 官方建議所產生的 hs38DH.fa 和來自 GATK 的Homo_sapiens_assembly38.fasta contig number （3366）相同，但檔案格式不同，在 HLA typing 的測試中，Kourami 和 Hisat2 兩個軟體在來自 bwa-kit 的 hs38DH.fa 皆有較好的 performance，而來自 bwa-kit 的 hs38DH.fa 可以自行更新，因此所產生的 contig number 會隨之而改變。  


## Reference
> https://gatk.broadinstitute.org/hc/en-us/articles/360035890711-GRCh37-hg19-b37-humanG1Kv37-Human-Reference-Discrepancies.
> 
> https://www.ncbi.nlm.nih.gov/grc
> 
> https://www.ncbi.nlm.nih.gov/grc/help/definitions/
> 
> https://gatk.broadinstitute.org/hc/en-us/articles/360035890711-GRCh37-hg19-b37-humanG1Kv37-Human-Reference-Discrepancies#comparison
> 
> https://cloud.google.com/life-sciences/docs/resources/public-datasets/reference-genome
> 
> Yan Guo, Yulin Dai, Hui Yu, Shilin Zhao, David C. Samuels, Yu Shyr,Improvements and impacts of GRCh38 human reference on high throughput sequencing data analysis,
Genomics,Volume 109, Issue 2,2017,83-90,ISSN 0888-7543,
https://doi.org/10.1016/j.ygeno.2017.01.005.
