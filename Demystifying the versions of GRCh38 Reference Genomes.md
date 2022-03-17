# Demystifying the versions of GRCh38/hg38 Reference Genomes, how they are used in DRAGEN™ and their impact on accuracy
原文網址：https://www.illumina.com/science/genomics-research/articles/dragen-demystifying-reference-genomes.html  
原文作者：Serverine Catreux, Fred Farrell, Rami Mehio, Lisa Murray, Gavin Parnaby, Cooper Roddey, Mike Ruehle  

譯者：陳品瑄，簡培苗，許書睿  
全文翻譯經過illumina作者授權。  

## 大綱 (Abstract)
  進行序列比對和變異位點偵測時，選擇合適的參考序列對分析結果的準確性有直接影響。這項決策是選擇GRCh37/hg19或GRCh38/hg38版本作為參考序列的延伸。版本之間存在不同差異的地方包含：是否涵蓋ALT [註一] 或是Decoy [註二] 序列以及涵蓋的序列範圍。最終的版本選擇將影響變異偵測的能力，可以從最近*CBS*, *CRYAA*和*KCNE1*幾個基因的實例說明：在GRCh37/hg19版的參考序列中，三者具有正常的一對基因資訊(one copy of the gene)，但在GRCh38/hg38版具有偽重複的基因資訊(false duplication)；使得定序資料比對至參考序列時，產生模糊的位置比對結果。 因此Genome in a Bottle Consortium (GIAB) 在2021年提出，使用「遮蓋(masking)」的方式移除偽重複序列，來有效地減少重複資訊對定序處理及變異準確率的影響。[1]  
  
DRAGEN團隊在2020年時，使用另一種改善變異位點偵測準確率的替代方法。DRAGEN開發了一個圖形化基因體(graph-based genome) [註三] 改善了Illumina定序出的片段在「基因體序列難以比對區域(Difficult-to-Map Regions)」的比對正確率。此技術的改善可從變異偵測的結果獲得確認。先前PrecisionFDA Truth Challenge v2 [2] 競賽中，在基因體序列難以比對區域(Difficult-to-Map Regions) [註四] 及其他全部基準區域(All Benchmark Regions) [註五] 的分析結果中，獲得最佳準確率的殊榮。圖形化基因體(graph genome)是在原本的GRCh38/hg38版序列中，增加了成千上百條從族群單倍體分型(phased population haplotype) [註六] 得到的ALT contigs。這些資訊的加成，等於是產生另一版的GRCh38/hg38參考序列。  

藉由增加參考序列資訊來改善變異點位偵測準確率，同時不影響後續註釋(annotation)的策略是相當有利的。特別是增進資訊的序列區域中，包含許多臨床檢測相關的基因位點。然而，我們也意識到每當增加具有新特點的參考序列時，常讓使用者產生疑惑。因為GRCh38/hg38的這個名稱，已不再能完整地描述研究中所使用的參考序列特性。本篇文章希望能澄清具有不同的特性的GRCh38/hg38版序列、同時描述目前我們推薦在DRAGEN平台上使用的參考序列，並討論後續對參考序列的改善策略。  

## 推薦的GRCh38/hg38版本 (Recommended GRCh38/hg38 Reference Versions)

現有兩種GRCh38/hg38版的參考序列，被DRAGEN3.9版推薦使用於分析中:hg38-alt-masked (non-graph)和hg38-alt-masked-graph，兩種目前皆可供下載 [3] 。兩參考序列皆有遮蓋ALT contigs資訊的功能，並可在DRAGEN v3.9版使用。這項功能比起先前使用座標轉換(liftover-based) [註七] 來處理ALT contigs資訊的參考序列，如hg38-alt-aware (non-graph)和hg38-alt-aware-graph，稍微改善定序準確率。下表1列出這四種檔案的分類和不同DRAGEN版本使用的主要序列：  

![](https://i.imgur.com/gcJCATy.jpg)
表1：DRAGEN推薦使用的GRCh38/hg38版參考序列。這四個版本的基礎序列可從註解 [4] 的網址下載。  


目前 hg38-alt-masked參考序列可供下載 [3] 。序列的hash table可由下列指令建立。  
```
dragen --ht-reference hg38.fa --ht-alt-aware-validate=true --ht-num-threads=40 --build-hash-table=true --ht-build-rna-hashtable=true --enable-cnv=true --ht-mask-bed /opt/edico/fasta_mask/hg38_alt_mask.bed --output-directory hg38_alt_masked
```
建立alt-masked hash table的功能從DRAGEN 3.9版開始提供，但此檔案可使用在舊版DRAGEN。  

hg38-alt-masked-graph hash table目前可供下載 [3] ，此檔案和3.9版前的DRAGEN可相容。但DRAGEN不提供使用者自行建立客製化的圖形化基因體；如果任意更改族群單倍體資料，可能會造成單倍體資訊和序列資訊發生矛盾，造成比對準確率下降。因此在建立圖形化基因體需要考量單倍體資訊的一致性，而目前DRAGEN尚未提供自動建立圖形化基因的功能。  

值得注意的是：進行序列比對產生之BAM檔與進行變異偵測的相容性，兩步驟使用的參考序列必須是一樣的。因此假設BAM檔是以alt-aware作為參考序列，下一步的變異偵測也必須使用同樣的alt-aware檔案。若使用者想將參考序列更新成alt-masked版的話，則建議以alt-masked作為參考序列，重新生成BAM檔後再進行變異偵測。  

## 所有GRCh38/hg38版本共有的基礎序列 (Common Reference Assembly as the baseline to all GRCh38/hg38 versions)

所有DRAGEN推薦給用戶使用的GRCh38/hg38版參考序列，都是以一共同的FASTA檔為基礎組裝而成 [4] 。子版本間的差異，是使用不同的族群單核苷酸多態性(SingleNucleotide Polymorphism，簡稱SNP)及族群單倍體資料進行資訊擴增，提供使用者線性參考序列(linear reference) [註3] 外的另一種選擇，並被稱作圖形化基因體(graph-based genome) [2] 。此外，原本GRCh38/hg38版中的ALT contigs和基本序列，都可經由DRAGEN mapper進行 alt-awareness 或是 alt-masked的功能進行處理。  

GRCh38/hg38常使用的FASTA序列包含3,366個contigs，全部的種類及數量列在下表2中：  

![](https://i.imgur.com/NMkstXd.jpg)
表2：常用的FASTA檔序列組成，可從註解 [4] 的網址下載。  


## 處理原本GRCh38/hg38版的ALT contigs的方法：alt-aware vs. alt-masking (Handling of native GRCh38/hg38 ALT contigs: ALT-aware versus ALT-masking)
在3.9版釋出前，DRAGEN利用ALT-aware alignment liftover的方式處理在hg19或hg38版FASTA檔內的ALT contigs，這個方法使用一份由Genome Reference Consortium產生的SAM檔，檔案記錄了ALT contigs比對回參考序列時，其相對應的染色體及位置座標。因此當序列和ALT contig比對得上，且此ALT contig在liftover SAM檔中具有明確的座標資訊時，此序列會被視作比對成功，並對到參考序列上相應的染色體區域；若序列比對到沒有座標資訊的ALT contig，則仍被算在和ALT contigs比對上的類別。  

![](https://i.imgur.com/dZkSoMM.jpg)
圖1：ALT區域和主要參考序列進行正確座標轉換的例子。


DRAGEN利用座標轉換處理ALT contigs的方法大幅增進了分析準確率，但隨測試進展，研究者發現一難以處理的問題：當比對的序列長度大於5Mb時，其比對和座標轉換結果可信度低。參考序列上許多區域，在長片段ALT haplotype和基礎序列(primary assembly)的座標比對上具有模糊性。不正確的座標轉換資訊，會生成許多不正確的位置比對結果、並影響變異偵測。目前，時不時有錯誤的座標轉換影響序列比對和變異偵測的問題發生，雖影響範圍局部但具有嚴重性。找出錯誤的座標轉換區域是相當費心力的過程。有鑒於問題出在ALT-aware的方法，我們發展了一套優於目前處理ALT contig的方案。  

![](https://i.imgur.com/wyfhmfs.jpg)
圖2：錯誤的座標轉換與比對將會產生額外的FP資訊。


DRAGEN開發了一套新方法來處理參考序列內原有的ALT contgis：透過遮蓋(masking)來提升準確率。和基礎序列(primary assembly) 座標相似度高的ALT片段將會被遮蓋，以減少和原有序列發生競爭、造成序列比對到錯誤座標的問題，同時提升序列比對的品質；若ALT片段和基礎序列相似度低則不會被遮蓋，通常屬於decoy sequence這類的片段。某些區域的邊界序列相似，也會使用遮蓋來提升比對準確率。如同圖三的結果分析，使用DRAGEN遮蓋ALT contigs後的參考序列、比起使用座標轉換處理ALT contigs的參考序列，可提升變異偵測的準確率。遮蓋處理的優勢在於，使用ALT contigs資訊的同時、排除影響結果的片段，且較容易進行功能更新和維護。未來遮蓋功能將會持續修正，但整體表現已超越座標轉換的處理方法。  

DRAGMAP，一款由DARGEN和Broad Institute共同開發的開源序列比對軟體(https://github.com/Illumina/DRAGMAP) 目前不支持ALT-awareness版本的參考序列 [5]。相反地，DRAGENMAP推薦使用alt-masked版本的參考序列，和最新版DRAGEN使用相同版本，這代表遮蓋法比起先前的座標轉換方法，在分析準確率仍有改進及發展空間，但已略勝舊有的處理方法。  

## 圖形化參考序列 (Graph Based Reference)
前面提到DRAGEN也支持使用圖形化基因體，來提升序列在基因體序列難以比對區域的準確率。圖形化功能不使用GRCh38/hg38版中原有的ALT contigs，而是選擇經同源片段比對的族群單倍體序列，來擴充ALT資訊。  

序列比對困難的原因在於：若某區域的序列多樣性高（如免疫MHC區域）會使序列歧異度高，造成比對軟體無法準確判斷對應至參考序列的位置。最常見的情況是：某序列可對應到參考序列上的A區、但同時也可比對到B區-通常是兩區域具有相似的重複片段(segmental duplication, highly repetitive sequences)-這時比對至兩區域的準確度相當，軟體也難以分辨。  
 
大部分比對困難的情況，可藉由族群內已知的基因變異模式，來排除相似序列的影響。假設一段序列可以比對至區域A和區域B，雖在兩區域的比對準確度相同，但都各具有兩個和參考序列不同的鹼基差異。根據比對軟體的設定，一般會將序列隨機比對至A或B區域，並給予低比對品質(MAPQ=0)；但如果我們知道序列和A區域的鹼基差異，是屬於族群內常見的變異，而和B區域的鹼基差異在族群中並不常見時，就可在族群基因體的知識下，將序列比對到可信度較高的A區域。  

族群單倍體基因的選擇是改善序列比對準確度的關鍵。增進族群序列的總數和多樣性雖然可改善比對準確度，但若族群單倍體資料不一致，也可能導致比對結果更加模糊並降低準確度。  

遮蓋處理的hg38圖形化參考序列同樣擁有較好的表現。目前圖形化參考序列只有在以硬體加速的DRAGEN平台提供，DRAGMAP軟體尚未提供使用。  

## 評估參考序列的表現 (Measuring the performance of the references)

使用遮蓋處理(alr-masked)及座標轉換處理(alt-awareness)的參考序列、對分析結果的影響如下圖三。兩方法又可分成非圖形化及圖形化參考序列共四種類型，並以NISTv.4.2.1 [註八] 當作標準答案 [6] 。可發現以遮蓋法處理參考序列的組別，在兩種主要變異種類SNP,INDEL的偵測上，和標準變異組相比皆有較少的FP+FN數量。  

![](https://i.imgur.com/HFHxAIj.jpg)
圖3：以NIST 4.2.1版作標準答案，比較使用alt-masked和alt-aware版參考序列的準確率差異。  


使用圖形化及非圖形化參考序列的變異偵測結果比較如圖四(SNP)及圖五(INDEL)所示。以PrecisionFDA v2 Challenge競賽中使用的NIST v.4.2.1作為標準答案（包含紀錄變異位點的VCF及變異區域的BED檔案）相比於非圖化參考序列，DRAGEN圖形化序列的使用可減少～50%偵測SNP變異的錯誤率、以及～27%偵測INDEL的錯誤率。NIST也在競賽後，釋出了HG001-HG007共7個標準品的v.4.2.1版標準變異檔案。下圖也再次證實使用圖形化基因體對於變異偵測的改善、及使用機器學習搭配圖形化基因體偵測小片段變異的可行性。以上結果也顯示DRAGEN更新的圖形化參考序列並非針對特定樣本，有應用到其他樣本及族群的潛力。  

![](https://i.imgur.com/4fDQaWo.jpg)
圖4：HG001到HG007共7個樣本，在SNP種類的變異偵測結果（以 v.4.2.1的VCF和BED檔做標準答案）。

![](https://i.imgur.com/JlbHko2.jpg)
圖5：HG001到HG007共7個樣本，在INDEL種類的變異偵測結果（以 v.4.2.1的VCF和BED檔做標準答案）。


利用標準答案評估圖形化基因體表現時要注意，必須使用最新版的標準變異組作為答案。例如使用v.4.2版的標準答案來評估時，可發現DRAGEN圖形化序列的使用、可減少近50%的變異錯誤總數；但若使用v.3.3.2版來衡量表現則沒有明顯差異，不僅是因為v.3.3.2版沒有涵蓋基因體序列難以比對區域，也因為v.4.2.1版有修正先前版本的錯誤、可以提供更正確的資訊。確實，使用DRAGEN圖形化作為參考序列的分析結果、在使用v.3.3.2版評估結果時、比起使用新版評估，統計出更多的偽陽性變異(FP)，其中有一大部分是v.3.3.2版資訊不完整造成的。  

## 後續對於GRCh38/hg38版本的改進 (Further GRCh38/hg38 Reference Improvements adopted by DRAGEN beyond 3.9 release)

目前圖形化和遮蓋處理參考序列的方法仍未達到一定的準確度。Genome in a Bottle (GIAB) Consortium, Genome Reference Consortium (GRC) 和 Telomere-to-Telomere (T2T) Consortium等三個學術聯盟也致力於精進GRCh38/hg38版序列 [1] ，主要為兩種類的改善：(1)遮蓋偽重複序列；(2)增加新的Decoy contigs。DRAGEN正在評估這些改善的成效，並合併至最新版的參考序列。表3記錄DRAGEN未來將配合新版本的釋出來更新參考序列：首先將會引入遮蓋序列的功能，並將其命名為"alt-masked-V2"；而增加Decoy的改善尚未完成，預計在之後釋出。  

![](https://i.imgur.com/KW31jxc.jpg)
表3：DRAGEN推薦使用的GRCh38/hg38版參考序列，分成圖形化(graph)及非圖形化(non-graph)。這四個版本的基礎序列可從註解 [4] 的網址下載。


### 針對新版參考序列：alt-masked-V2的詳細說明 (More details on the newly masked bases: towards alt-masked-V2)
最近GIAB釋出一組移除偽重複序列的GRCh38/hg38版參考序列 [1] 。GIAB和GRC合作發展出一份記錄著GRCh38版中將被遮蓋的區域檔案，且保證遮蓋功能不會影響位置座標及變異偵測結果，因為遮蓋處理掉的都是不正確的序列或污染。這些重複區域是由Telomere-to-Telomere (T2T) 聯盟偵測到的。此外，被遮蓋處理的序列包含部分第21號染色體，主要是為了改善發育相關基因的比對正確率，如*CBS*, *CRYAA*和*KCNE1*等基因。  

圖6為參考文獻 [1] 的補充資料，資料顯示以GRCh37版作為參考序列、並使用不同定序技術處理時，對於*KCNE1B*基因變異的偵測結果具有高度一致性，這是因為GRCh37版序列並不包含偽重複序列。但以GRCh38版作為參考序列時，許多包含*KCNE1B*的序列會被比對至錯誤的序列上。  

![](https://i.imgur.com/B1mlKcm.jpg)
圖6：*KCNE*基因的覆蓋度根據使用的參考序列(GRCh37或GRCh38)而有所不同（且參考序列尚未使用遮蓋功能處理）。


下圖7和圖8評估GRCh38版經由遮蓋改善的結果，將7個標準品分別使用alt-masked 和 alt-masked-V2作為參考序列，並比較變異偵測的準確率(如錯誤偵測的變異總數、FP+FN數量等)分成SNP及INDEL兩種。  

![](https://i.imgur.com/addQN2a.jpg)
圖7：將7個標準品分別使用alt-masked 和 alt-masked-V2作為參考序列並以V.4.2.1 NIST作為標準答案，發現使用GIAB遮蓋處理的參考序列可改善準確度。  

![](https://i.imgur.com/s4Bti1v.jpg)
圖8：將7個標準品分別使用alt-masked 和 alt-masked-V2作為參考序列並以V.4.2.1 NIST作為標準答案，發現使用GIAB遮蓋處理的參考序列可改善準確度。

表4顯示使用GIAB遮蓋處理的參考序列來偵測三個和發育相關基因變異的結果。使用最新版的challenging medically-relevant genes (CMRG) [1] 作為參考答案，可發現偵測結果大幅改善，挽救了所有過去被判定為FN和FP的變異。  

![](https://i.imgur.com/HgynR8Q.jpg)
表4： HG002偵測SNP變異的準確度，使用GIAB遮蓋處理的參考序列後，準確度具有顯著改善。  


### 即將新增Decoy contigs參考序列：alt-masked-V3的詳細說明 (More details on the upcoming new Decoy contigs: towards alt-masked-V3)

第二種改進準確度的方法為使用decoy sequence。 GIAB 和 Baylor College of Medicine [8] 正致力於評估GRCh38/hg38版序列，經decoy sequence擴增後，能否改進原本被偽陰性訊號影響而比對錯誤的序列，例如：當參考序列有兩段同源且高相似度的區域時，另外一段很有可能會被當成偽複製片段而被遮蓋掉，造成比對錯誤。而Decoy序列可以幫助證明同源序列的存在，並減少錯誤比對導致的偽陽性(FP)資訊。  

第三種改善參考序列的方法是利用族群單倍體增加圖形化基因體的ALT contig數量。DRAGEN目前使用的圖形化基因體已經大幅改善準確度，若有機會增加族群單倍體資訊，則可讓序列涵蓋更多元的區域。最終期望能建立一不同族群使用也能具有最高準確度得圖形化基因序列，降低參考序列不通用的問題。  


## 建立客製化的遮蓋考序列 (Creating A Custom Masked Reference)

目前 hg38-alt-masked參考序列可供下載 [3] 。序列的hash table可由下列指令建立。  
```
dragen --ht-reference hg38.fa --ht-alt-aware-validate=true --ht-num-threads=40 --build-hash-table=true --ht-build-rna-hashtable=true --enable-cnv=true --ht-mask-bed /opt/edico/fasta_mask/hg38_alt_mask.bed --output-directory hg38_alt_masked
```
請注意從DRAGENv3.9版開始，在建立hash table的指令行中，如果沒有指定的座標轉換或遮蓋區域檔案(BED)，DRAGEN的預設為自動引入遮蓋區域檔(alt-masked.bed)來產生遮蓋版本(hg38-alt-masked or hg19-alt-masked)的參考序列。此外遮蓋功能對於GRCh37和hs37d5版參考序列沒有作用，因為這些版本原本就不包含ALT contigs。  

DRAGEN提供建立客製化遮蓋序列的功能，利用修改DRAGEN提供的hg38_alt_mask.bed或自行建立一份檔案。DRAGEN建立hash table的方式，是利用bed記載的區域資訊，把FASTA檔中同區域的序列轉成N，並提供給序列比對分析。如果記錄在bed檔的區域沒在FASTA檔沒有出現，DRAGEN會停止hash table的建立。在之後的版本，DRAGEN將不會暫停指令並只遮蓋FASTA檔中可對應上的區域。  

hg19版的注意事項：  
hg19版參考序列有包含一小部分的ALT contigs，所以針對hg19版使用alt-aware或 alt-masked處理的結果差異，和上面針對hg38版的討論是相同的。  

## 結果歸納和致謝 (Summary of Results and Acknowledgement)
從分析比對與偵測變異工具的準確度延伸至複雜的參考序列選擇，參考序列需要定期更新和調整，並釋出可以改善變異偵測與區域比對結果的新版本。文章中我們介紹了一部分DRAGEN3.9版參考序列使用的改善方法，以及後續的的改善方向。  

我們想致謝Genome in a Bottle (GIAB) Consortium, Genome Reference Consortium (GRC) 和 Telomere-to-Telomere (T2T) Consortium等致力於序列改善的學術聯盟，並成功增加比對等分析結果準確度。這樣的結果對臨床的基因檢測相當重要，給予醫藥決策與改善人類健康一大助力。  

## 原文註解
[1] Wagner et al. [Towards a Comprehensive Variation Benchmark for Challenging Medically-Relevant Autosomal Genes.](https://www.biorxiv.org/content/10.1101/2021.06.07.444885v3) BioRxiv (2021) doi:2021:09.07  
[2] [DRAGEN Wins at PrecisionFDA Truth Challenge V2 Showcase Accuracy Gains from Alt-aware Mapping and Graph Reference Genomes](https://www.illumina.com/science/genomics-research/articles/dragen-wins-precisionfda-challenge-accuracy-gains.html)  
[3]https://support.illumina.com/sequencing/sequencing_software/dragen-bio-it-platform/product_files.html  
[4]http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/GRCh38_reference_genome/GRCh38_full_analysis_set_plus_decoy_hla.fa  
[5]https://github.com/Illumina/DRAGMAP/  
[6] Wagner et al. [Benchmarking challenging small variants with linked and long reads.](https://www.biorxiv.org/content/10.1101/2020.07.24.212712v5) BioRxiv (2021) doi:2020:07.24  
[7] Nurk et al. [A complete reference genome improves analysis of human genetic variation.](https://www.biorxiv.org/content/10.1101/2021.05.26.445798v1) BioRxiv (2021) doi: 2021.05.26.445798  
[8] Publication pending.  

## 名詞解釋
[註ㄧ] ALT:DNA序列與主要染色體相似的片段，獨立出來陳列於參考序列reference genome。  
[註二] Decoy:已知無意義DNA序列，獨立出來陳列。可增加分析精準度。  
https://support.illumina.com/content/dam/illumina-support/help/Illumina_DRAGEN_Bio_IT_Platform_v3_7_1000000141465/Content/SW/Informatics/Dragen/HandlingDecoyContigs_fDG.htm  
[註三] graph-based and linear genome:圖形化及限線性化基因體的定義可參考此文獻。  
Ameur, A. [Goodbye reference, hello genome graphs.](https://www.nature.com/articles/s41587-019-0199-7) Nat Biotechnol 37, 866–868 (2019).  
[註四] Difficult-to-Map Regions:難以比對的原因為此區域常有重複或相似序列，使序列比對具有模糊性，軟體通常會將序列隨機比對至某區，並給予低比對品質的標記。  
[註五] All Benchmark Regions:全部基準區域是指出現在此區域內的變異都是經過驗證並真實存在的，即使改用不同分析工具，也可偵測得到此區域內的變異。  
[註六] phased population haplotype: 族群單倍體分型  
[註七] liftover-based：座標轉換可參考此文獻。  
Luu PL et al. [Benchmark study comparing liftover tools for genome conversion of epigenome sequencing data.](https://academic.oup.com/nargab/article/2/3/lqaa054/5881791) NAR Genom Bioinform. 2020;2(3):lqaa054.  
[註八] NISTv.4.2.1: 這是一組包含紀錄變異位點的VCF檔案及變異所在區域的BED檔案，通常作為標準答案來評估工具的準確度，檔案ftp連結如下。  
ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/release/
