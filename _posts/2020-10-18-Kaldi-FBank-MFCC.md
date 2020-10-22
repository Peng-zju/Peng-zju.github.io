---
published: true
---
## Kaldi Turorial - Computing Fbank and MFCC Features for Single Utterance

Suppose you have an utterance audio file _robin.flac_ and want to compute Fbank and MFCC features based on it.

There are two binaries in `kaldi/src/featbin/` you can use:
- **compute-fbank-feats** [options...] \<wav-rspecifier> \<feats-wspecifier>
- **compute-mfcc-feats** [options...] \<wav-rspecifier> \<feats-wspecifier>

### Step 1 - Add binary to PATH
Execute `path.sh` from any exmaple directory so that the above two binaries are in PATH.
  
`source kaldi/egs/mini_librispeech/s5/path.sh`

### Step 2 - Prepare wav.scp 
wav.scp is a [Kaldi script file](https://kaldi-asr.org/doc/io.html#io_sec_scp) that consists of mutiple lines of *\<key> \<rxfilename> pairs.

If audio is in *wav* format, then \<rxfilename> is the wav file location. For instance, in **yesno** example: 

`head -1 kaldi/egs/yesno/s5/data/train_yesno/wav.scp`
> 0_0_0_0_1_1_1_1 waves_yesno/0_0_0_0_1_1_1_1.wav 

If audio is in other format, like *flac*, then \<rxfilename> is a command  followed by a pipe `|` . For instance, in **mini_librispeech** example:

`head -1 kaldi/egs/mini_librispeech/s5/data/train_clean_5/wav.scp`
> 1088-134315-0000 flac -c -d -s ./corpus/LibriSpeech/train-clean-5/1088/134315/1088-134315-0000.flac |

In order to Fbank and MFCC accepts wav format. So modify the wav.scp by adding a [sox](http://sox.sourceforge.net/) pipe, saving this line to a seperate file to : `kaldi/egs/mini_librispeech/s5/wav_test.scp`
>1088-134315-0000  flac -d -s -c ./corpus/LibriSpeech/train-clean-5/1088/134315/1088-134315-0000.flac | **sox -t wav - -t wav - |**

### Step 3 - Compute Fbank
Use *1088-134315-0000.flac* from the mini_librispeech as an example. Output the feature to stdout. 

To see avaibable options, like `--help`, type the the command without any arguments:

`compute-fbank-feats`

Here we specify the number of fiterbanks to be 40. 

    cd kaldi/egs/mini_librispeech/s5/
    compute-fbank-feats --num-mel-bins=40 scp:'head -1 wav_test.scp |' ark,t:- | head
>1088-134315-0000  [
  6.028292 6.624897 7.830281 8.859182 10.41348 10.91126 11.25801 11.74803 12.88105 14.63195 14.06285 14.49745 14.63049 16.33435 16.58837 14.91046 14.31796 16.94013 17.48193 16.97827 17.59924 18.45235 16.66505 16.82691 15.69515 16.65768 17.22556 16.99131 16.60596 17.06459 16.78841 16.04231 15.69495 15.10274 15.19935 14.40812 14.33042 13.52844 12.31978 13.13282 ...
  
`ark, t` means write archine in text instead of binary and `-` means stdout. See [Kaldi I/O mechanisms](https://kaldi-asr.org/doc/io.html)

### Step 4 - Compute MFCC

To see avaibable options for compute-fbank-feats, type: 

`compute-fbank-feats`

Here we keep all options to default

    compute-mfcc-feats scp:'head -1 wavtest.scp |' ark,t:- | head
   >1088-134315-0000  [
  15.80592 -19.65983 -44.14267 -10.36022 -21.32767 -3.718761 -16.35019 -2.041183 1.383831 -6.504814 -15.53979 4.017059 -1.488521
  15.80938 -19.18601 -36.05476 1.974784 -16.47062 9.139902 -17.07959 -2.513923 12.35208 0.3189384 -16.05096 -10.61343 -5.74498 .....

### Additionals
Use `feat-to-len` to map utterance-id to utterance length in frames
Use `compute-cmvn-stats` cepstral mean and variance normalization, per-utterance by default, or per-speaker.

