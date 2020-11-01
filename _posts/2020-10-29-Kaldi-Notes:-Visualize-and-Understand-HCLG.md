---
layout: post
---

In this post, we will visualize each component of the HCLG by reviewing the building process of decoding graph for the `yesno` example located in `kaldi/egs/yesno/s5` directory.

The script `run.sh` contains pipelines to  build a complete mono-phone model including the GMM acoustic model, the HCLG decoding graph and decoding results using test data. 
First run the script. The script will also download and prepare the corpus and language related data in `data/lang`.

`$ cd kaldi/egs/yesno/s5`

`$ source run.sh`

<br>
<hr />

## G.fst

G in HCLG represents the language model. Both input label and output label are word-id. First let's take a look at the given LM in ARPA format:

`$  less input/task.arpabo`

>\data\\ <br>
>ngram 1=4
>
>\1-grams:<br>
>-1	NO<br>
>-1	YES<br>
>-99 \<s><br>
>-1 \</s>
>
>\end\

The `run.sh` scirpt calls the `local/prepare_lm.sh`, which uses the `arpa2fst` binary script to generate G.fst based on the LM: 

>`$ arpa2fst --disambig-symbol=#0 --read-symbol-table=$test/words.txt input/task.arpabo $test/G.fst`

Use the following command to visualize G.fst. You may have to install other dependancies according to your program output.

`$ cd data/lang_test_tg/`

`$ fstdraw --isymbols=words.txt --osymbols=words.txt G.fst | dot -Tjpg >  G.jpg`

<br>

![G.fst](../images/2020-10-29-G-fst.jpg)

To check the word-id for each word:

`$ less data/lang_test_tg/words.txt`

>\<eps> 0 <br>
>\<SIL> 1 <br>
>NO 2 <br>
>YES 3 <br>
>#0 4 <br>
>\<s> 5 <br>
>\</s> 6

<br>
<hr />

## L.fst

L in HCLG represents the lexicon. The input lable is phone, and output label is word. Kaldi used `utils/prepare_lang.sh` to prepare L.fst (used for training) and L_disambig.fst (used for decoding). 

To check how many phones we have:

`$ less phones.txt`

><eps> 0<br>
>SIL 1<br>
>Y 2<br>
>N 3<br>
>#0 4<br>
>#1 5

Then take a look at the L_disambig.fst:

`$ fstdraw --isymbols=phones.txt --osymbols=words.txt L_disambig.fst | dot -Tjpg >  L_disambig.jpg`

<br>

![L.fst](../images/2020-10-29-L-fst.jpg)

Given the L_disambig.fst, say we have a phone input sequence 

> \<eps> SIL #1 Y 

Then the corresponding word output sequence is 

> \<eps> \<eps> \<eps> YES 

<br>
<hr />

## LG.fst

Now we compose L_disambig.fst and G.fst into LG.fst. The input label is phone and output lable is word. Run the following command to generate LG.fst (actually, run.sh has generated LG.fst under `data/lang_test_tg/tmp`):

`$ fsttablecompose L_disambig.fst G.fst | fstdeterminizestar --use-log=true | fstminimizeencoded | fstpushspecial > LG.fst`

Then visualize LG.fst:

`$ fstdraw --isymbols=phones.txt --osymbols=words.txt LG.fst | dot -Tjpg >  LG.jpg`

<br>

![LG.fst](../images/2020-10-29-LG-fst.jpg)

<br>
<hr />

## CLG.fst

C in HCLG means fst for context dependent phone. Its input is cd-phone, output is phone. Kaldi doesn't generate C.fst separately, then compose with LG. Instead, it dynamically generate the context FST and compose with LG to produce CLG.fst using the `fstcomposecontext` binary.

The `run.sh` script trains a mono-phone system. So the phones are not context dependent.

<br>
<hr />

## H.fst

H in HCLG means fst for HMM states. Its input is transition-id (which can be mapped to phone, hmm-state, and pdf-id). Its output is cd-phone. In Kaldi, you can use binary `make-h-transducer` to create H.fst. 

`$ cd ../..`

`$ make-h-transducer tmp/ilabels_1_0 ../../exp/mono0a/tree ../../exp/mono0a/final.mdl > H.fst`

Then visualize H.fst:

`$ fstdraw  H.fst | dot -Tjpg >  H.jpg`

<br>

![LG.fst](../images/2020-10-29-H-fst.jpg)

The input label ranges from 0 to 31, corresponding to the transition-id. Now take a look at what each transition-id means:

`$ show-transitions phones.txt ../../exp/mono0a/final.mdl`

>Transition-state 1: phone = SIL hmm-state = 0 pdf = 0<br>
 Transition-id = 1 p = 0.73957 [self-loop]<br>
 Transition-id = 2 p = 0.01 [0 -> 1]<br>
 Transition-id = 3 p = 0.12042 [0 -> 2]<br>
 Transition-id = 4 p = 0.130011 [0 -> 3]<br>
...<br>
Transition-state 6: phone = Y hmm-state = 0 pdf = 5<br>
 Transition-id = 19 p = 0.709512 [self-loop]<br>
 Transition-id = 20 p = 0.290488 [0 -> 1]<br>
 ...<br>
Transition-state 9: phone = N hmm-state = 0 pdf = 8<br>
 Transition-id = 25 p = 0.872881 [self-loop]<br>
 Transition-id = 26 p = 0.127119 [0 -> 1]<br>
 ...

 Generate an ilabel text file to re-draw the H.fst. Each line in H_fst_ilabel.txt lists the text label for each ilabel.:

 >\<eps> 0<br>
SIL-1 1<br>
...<br>
Y-19 19<br>
...<br>
N-25 25
..<br>
31 31<br>

`$ fstdraw --isymbols=Ha_fst_ilabel.txt --osymbols=phones.txt H.fst | dot -Tpdf >  H.pdf`

<br>

![LG.fst](../images/2020-10-29-H-fst-1.png)

Notice there is a loop 0-7-8-9-0, in which the ilabel is Y-20 Y:22 Y:24. This stands for the 3 HMM states of phone "Y".

The loop 0-10-11-12-0, in which the ilabel is N-26 N-28 N-30. This stands for the 3 HMM states of phone "N"

<br>
<hr />

## HCLG.fst

The ilabel in HCLG is HMM-state, and olabel is word. The `run.sh` has generated the HCLG.fst when making decoding graph as in `exp/mono0a/graph_tgpr/HCLG.fst`.

`$ fstdraw --isymbols=Ha_fst_ilabel.txt --osymbols=words.txt ../../exp/mono0a/graph_tgpr/HCLG.fst | dot -Tpdf >  HCLG.pdf`

<br>

![LG.fst](../images/2020-10-29-HCLG-fst.png)

From starting state 0 to ending state 9, let's generate a state path:

>0 10 10 1 13 0 12 12 12 8 9 3 3 6 6 20 20 7

The correponding HMM state ilabel path is:

>SIL-6 SIL-5 \<eps> SIL-9 \<eps> SIL-8 SIL-5 SIL-5 \<eps> SIL-18 Y-20 Y-19 Y-22 Y-21 Y-24 Y-23 \<eps> 

The corresponding word olabel path is:

>\<eps> \<eps> \<eps> \<eps> \<eps> \<eps> \<eps> \<eps> \<eps> \<eps> YES \<eps> \<eps> \<eps> \<eps> \<eps> \<eps> 


