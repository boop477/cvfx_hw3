# 【CV_hw3】GAN-Dissection

## Outline
* [Goal](#Goal)
* [GAN Dissection](#GAN-Dissection)
    * [Generate images with GANPaint](#Generate-images-with-GANPaint)
    * [Dissect any GAN model and analyze](#Dissect-any-GAN-model-and-analyze)
* [Compare with other methods](#Compare-with-other-methods) 
  * [Exemplar-Based Inpainting](#Exemplar-Based-Inpainting)
  * [Globally and Locally Consistent Image Completion](#Globally-and-Locally-Consistent-Image-Completion)

* [Conclusion](#Conclusion)
* [Appendix](#Appendix)
* [Reference](#Reference)

## Goal
根據使用者的輸入移除或生成在自然影像中的物件。

## GAN Dissection
Gan Dissection試著解決GAN model中學到black box的問題，透過分析並且視覺化不同layer中的unit去了解GAN model在不同層數時學到的東西。Gan Dissection從兩個面向分析不同unit:

**1. Dissection**: 評估unit所著重的class。GAN Dissection透過計算`intersection-over-union(IoU)`去評估unit和所有class之間的相似性，並且將相似性最高的class當作該unit所著重的class。

**2. Intervention**: 找出和生成/移除target cass有關的一組units。GAN Dissection計算`Average causal effect`找出和class有關的uints.

找出`Dissection`，`Intervention`之後我們可以透過控制和某一個class相關的unit來生成或是移除該class。

### Generate images with GANPaint
| | Generated Image |
|:-------:| :-------:| 
| （a) Original |![](https://i.imgur.com/bWGlSC6.jpg)|
| (b) Remove tree |![](https://i.imgur.com/xOvCab2.jpg)|
| ( c) Remove glass |![](https://i.imgur.com/x5uK5kZ.jpg)|
| (d) Remove Door |![](https://i.imgur.com/KLUaHHl.jpg)|
| (e) Add cloud to the sky |![](https://i.imgur.com/6RY7I9p.jpg)|
| (f) Add cloud to the whole image |![](https://i.imgur.com/r6xMrKw.jpg)|

*<p align="center">Fig 1. 使用Intervention改變生成圖片內容。</p>*

&emsp;&emsp;我們從生成圖片觀察到，在合理的改變之下可以生成合理的圖片。例如將樹木移除後，生成的圖片變成建築物(Fig 1b)；將房子前的草地移除後則是變成人行道(Fig 1c)；把門移除的話則是產生窗戶(Fig 1d)。此外我們也在天空中加上雲，生成的圖片可以在原本是天空的地區生成雲，但是有些塗到建築物的地方則是會讓建築物的輪廓變形(Fig 1e 裡面圖片右側的尖塔變成紅蘿蔔形狀的謎樣物體)；如果我們把讓整張圖片都變成雲的話則會把原本平坦的地方(地面)試圖鋪上雲的texture，建築物的部分texture看起來不自然(Fig 1f)。<br>
&emsp;&emsp;從在不合理的地方生成雲這項操作中可以發現：model並非像複製貼上一樣在任何地方都可以生成物件，而是在經過learning之後學的該物件可能出現的合理地方才能夠生成物件。如果很任性地堅持要在不合理地方生成物件的話，除了無法生成之外更會讓原本就在那邊的變形或是失去自然的texture。

---
### Dissect any GAN model and analyze
| Layers | Visualized units | Visualized units | Visualized units |
|:-------:|:-------:| :-------:| :-------:| 
| Layer 1 |![](https://i.imgur.com/4Iw8R2v.jpg)|![](https://i.imgur.com/38akNxy.jpg)|![](https://i.imgur.com/RNBmmVR.jpg)|
| Layer 4 |![](https://i.imgur.com/pWoXIvC.jpg)|![](https://i.imgur.com/KVx4LkO.jpg)|![](https://i.imgur.com/LnvqXNK.jpg)|
| Layer 7 |![](https://i.imgur.com/YRZttiM.jpg)|![](https://i.imgur.com/jRaefdB.jpg)|![](https://i.imgur.com/AIQALaL.jpg)|

*<p align="center">Fig 2. Visualize units in layer 1, layer 4, and layer 7.</p>*

&emsp;&emsp;我們使用LSUN living room progressive GAN作為dissection target並且視覺化layer1 layer3 layer7裡面unit所代表的class。Fig 2內被黃線圈起來的區域代表該unit注意的class，我們可以發現訓練在living room的model裡面的unit會專注於living room常出現的傢俱上，像是書櫃，
落地窗，沙發和壁畫。


## Compare with other methods
### Exemplar-Based Inpainting

&emsp;&emsp;Exemplar-Based Inpainting算是image inpainting裡面比較傳統的方法。結合了texture synthesis與基於微積分的計算來實現目標的移除或是修復。
  
**Steps：**
1.   區域劃分
  *Fig.3-a*：Φ為來源區域，Ω為目標區域。
  *Fig.3-b*：p點為邊界上的某一點，以p點為中心設置一個區塊（綠色方框）。
  *Fig.3-c*：從來源區域找與p的為中心的區塊相似的區塊（q'和q''）。
  *Fig.3-d*：在相似區塊中找到更為適合匹配的區塊進行填充（優先級更高的區塊）。<br>
![](https://i.imgur.com/5ippERI.png)
*<p align="center">Fig.3 Structure propagation by exemplar-based texture synthesis </p>*

2.   找初始來源區域與目標區域的邊界
3.   Iteration（Repeat until done）： 
    
      *  計算以邊界區域像素點為中心的patch的優先級，以較強的邊緣連續區域與被高置信度的像素包圍的區域為優先。<br>
       Priority P(**p**) definition:<br>
       ![](https://i.imgur.com/0wH9Ipt.png)<br>
       C(**p**):confidence term:<br>
       ![](https://i.imgur.com/AFMMzpa.png)<br>
       I：Entire image
       Ψ<sub>p</sub>:待填充的patch（|Ψ<sub>p</sub>|:待填充patch的面積）
       Ω:Target region<br>
       D(**p**):Data term:<br>
       ![](https://i.imgur.com/F7jV2fS.png)<br>
       ∇I⊥p: isophote (direction and intensity) of p.
       n<sub>p</sub>:輪廓在p點的法向量
      ![](https://i.imgur.com/qczgIAv.png)<br>
      *<p align="center">Fig.4 Notation diagram of formula</p>*
       
      *  找到最高置信度的patch，基於該patch在來源區域尋找匹配的區域進行填充。
      *  將找到的最佳匹配區域複製至其對應patch。
      *  更新置信度(C(**p**))。
  
  <br>**Results:**
  
  Data source：[The Street View Text Dataset（SVT）](http://www.iapr-tc11.org/mediawiki/index.php/The_Street_View_Text_Dataset?fbclid=IwAR1hVNFBwjR1-a_fu2N769q549HRNPAqh5shCLrHj3h4_dgYwTTmqYPRk4E)
  
&emsp;&emsp;如圖*Fig.5*所示，第一個column是原圖，第二個column是手動mask（黑色部分為mask的區域）想要移除的人或物（分別是兩張人、花壇和窗戶），第三個column則是移除後的效果。

![](https://i.imgur.com/iLstnt4.jpg)
*<p align="center">Fig 5.  Left:Original Image  Center:Masked  Image Right:Output</p>*

<br>Pros：<br>
&emsp;&emsp;可以很明顯的看出，Exemplar-Based Inpainting在小範圍且mask邊緣背景顏色變化不大的區域移除效果不錯。

Cons：
1.   由於其計算並補齊缺失部分的過程中，修復出來的像素點的置信度逐漸降低，導致最後修復的部分會出現越來越模糊的結果，不適用與大面積的移除或者修復。
2.   由於其填補是以周圍像素作為來源區域，所以也不適用於邊界顏色複雜或是邊界顏色差異較大的區域。
  


---

### Globally and Locally Consistent Image Completion

&emsp;&emsp;Globally and Locally Consistent Image Completion(GL)是來自于日本早稻田大學的Satoshi Iizuka等人2017年發表于SIGGRAPH的文章。其論文提出了一個基於GAN思想建立的捲積網絡，重點在於其設計了兩種discriminator(global discriminator&local discriminator),以達到生成圖像能保證全局語義又能提高修復部分圖像的紋理質量和解析度。
<br>**Step:**
![](https://i.imgur.com/JWR6CzZ.png)
*<p align="center">Fig 6.Architecture for learning image completion</p>*
1. completion network<br>comcompletion network采用context encode的架構，包含encoding跟decoding兩個部分。encoding部分使用12層的convolution layers降低圖片的解析度到1/16的大小，爲了保證生成圖片不過於模糊，其中只使用了兩層的strided convolution將圖片解析度降低，并使用dilated convolutional layers（空洞捲積）能在相同的參數與計算能力下獲取更多的圖像信息。<br>
![](https://i.imgur.com/XvRkWIg.png)
*<p align="center">Fig 7. Architecture of the image completion network</p>*

2. Context discriminator<br>discriminator的部分如上所述分爲global discriminator與local discriminator，兩者分別通過了5x5的convolution layer和2x2的strid，最後通過全連接層連接使用sigmoid進行檢查（Real or Fake）來優化生成圖片。
![](https://i.imgur.com/HbxA8Sz.png)
*<p align="center">Fig 8.Architecture of the dsiscriminator</p>*
3. Post-processing <br>最後作者還增加了一個簡單的post-processing收尾，目的是修正有的情況下修補區域會與周邊環境色調上的較大誤差。
![](https://i.imgur.com/tUASxEC.png)
*<p align="center">Fig 9.Effect of simple Post-processing in GL</p>*
<br>**Result:**

| Source Image|Mask Image |Output Image |
|:-------:|:----------:|:------:|
|![](https://i.imgur.com/yu4Ag7y.png)|![](https://i.imgur.com/AhB9Ajw.png)|![](https://i.imgur.com/aQ5BVTX.png)|
|![](https://i.imgur.com/Rr0e7Lp.png)|![](https://i.imgur.com/duVJqln.png)|![](https://i.imgur.com/qzrMkxw.png)|
|![](https://i.imgur.com/FifoBrU.png)|![](https://i.imgur.com/deoMBYU.png)|![](https://i.imgur.com/sxRApw7.png)|

*<p align="center">Fig 10.Final works presentation</p>*

&emsp;&emsp;在成果測試時使用的Source images是取自GAN-Dissection，目的是爲了與後者做一個比對。
有上面的結果圖我們可以從第一組圖中看出，Iizuka的Inpainting在對物體邊緣區域缺失的case下實現的是removal的效果；
第二組圖片中我們扣掉的是建築物半包裹的一整棵綠色植物，最終呈現的是remove掉綠色的植物并且按照圖中建築的特色來實現缺陷部分的inpainting；
第三組圖片我們扣掉了半個屋頂，很有意思的是inpainting后生成的圖片出現了原圖未有的疑似烟囪的物體，這應該是其model訓練使用的是擁有1400w+images、2w+tags這樣龐大dataset的imagenet數據集，使得其能夠根據圖片整體信息生成出更符合實際生活的images。

| Source Image|Mask Image |Output Image |After Post-processing|
|:-------:|:----------:|:------:|:------:|
|![](https://i.imgur.com/guR9chL.png)|![](https://i.imgur.com/LOVH2UY.png)|![](https://i.imgur.com/8JQiEeJ.png)|![](https://i.imgur.com/4f1ogGf.png)|
|![](https://i.imgur.com/TDg8UnB.png)|![](https://i.imgur.com/t3s1kum.png)|![](https://i.imgur.com/QB001YI.png)|![](https://i.imgur.com/qPgJiXp.png)|
|![](https://i.imgur.com/hRPuBQB.png)|![](https://i.imgur.com/AfQrXIY.png)|![](https://i.imgur.com/ZhqcEgG.png)|![](https://i.imgur.com/zQdAjUF.png)|
|![](https://i.imgur.com/3WE1a92.png)|![](https://i.imgur.com/xTM6s6L.png)|![](https://i.imgur.com/dMSWJr4.png)|![](https://i.imgur.com/hnNkTii.png)|

*<p align="center">Fig 11.Final works with Post-processing presentation</p>*

&emsp;&emsp;我們也做了有無Post-processing之間的一個比較，可以發現效果並沒有論文中show出（詳見Fig.9）的，可能是因爲其采用的是比較基礎的平滑化處理,在缺失區域周圍較爲複雜的case下難以達到好的效果。

Cons：
經過與GAN-Dissection的對比，我們發現兩種做法在應用上都能夠實現圖像object的removal和inpainting，但是相比之下，Iizuka的inpainting在效果上并沒有GAN-Dissection好，在圖中各種object的feature的挖掘上并沒有GAN-Dissection徹底，缺失部分的inpainting雖有較高的解析度但是并沒有和其周邊的像素銜接的很好，也正是因爲這樣，作者不得不在最後加上一個用於平滑化的Post-processing。但也如上測試，感覺效果不怎麽樣。
> Although our network model can plausibly fill missing regions, sometimes the generated area has subtle color inconsistencies with the surrounding regions. To avoid this, we perform simple post-processing by blending the completed region with the color of the surrounding pixels. In particular, we employ the fast marching method [Telea 2004], followed by Poisson image blending [Pérez et al. 2003].
> [name=Sakoto Iizuka] 


  
## Conclusion
**GAN Dissection**分析不同layer中的unit和class的相關性(Dissection)並且找出一組和class的生成或消失的units(Intervention)。之後透過這兩個性質控制units並在畫面中生成或是移除物件，然而model在生成或是移除移除物件時仍然會用model學到關於該物件的合理性作為參考。
<br>**Exemplar-Based Inpainting**通過迭代更新像素點的置信度來進行移除或是修補圖像的效果。不用訓練，轉換速度快，在一定的環境下轉換效果尚可，但是對於較大的圖片或是高解析度的圖片可能效率很低甚至無法進行移除或修補。

## Appendix
![](https://i.imgur.com/ABNB5uM.jpg)

## Reference
[1] A. Criminisi, P. Perez, and K. Toyama, “Region filling and object removal by exemplar-based image inpainting,” IEEE Trans. Image Process., vol. 13, no. 9, Sep. 2004.<br>
[2] Satoshi Iizuka, Edgar Simo-Serra, and Hiroshi Ishikawa，"Globally and Locally Consistent Image Completion" ACM Transaction on Graphics (Proc. of SIGGRAPH 2017), 2017<br>
[3] D. Bau, J.-Y. Zhu, H. Strobelt, B. Zhou, J. B. Tenenbaum, W. T. Freeman, and A. Torralba, “Gan dissection: Visualizing and understanding generative adversarial networks,” arXiv preprint arXiv:1811.10597, 2018.
