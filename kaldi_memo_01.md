
## 음성인식 메모(kaldi) 1 - feature

http://work-in-progress.hatenablog.com/entry/2018/02/17/102153

음성인식 toolkit 'kaldi'를 시험해본다.

오늘은 feature extraction.

음성 데이터는 HTK의 HCopy로 해봤을때와 같은것을 사용.

kaldi 공식 사이트에는 다음과 같은 설명이 있는데, (HTK와는) 완전히 같게 되진 않는 것 같다.

>With the option –htk-compat=true, and setting parameters correctly, it is possible to get very close to HTK features.

일단은, wav 데이터를 아카이브 형식으로 저장.

>featbin/extract-segments scp:mosi1.scp mosi1_segment ark:mosi1.ark

이어서, MFCC feature를 추출.

>featbin/compute-mfcc-feats --config=mfcc.conf ark:mosi1.ark ark:mosi1.mfcc

### default option (MFCC)

（feat/feature-mfcc.h、struct MfccOptions）
```cpp
struct MfccOptions {
    FrameExtractionOptions frame_opts;
    MelBanksOptions mel_opts;
    （ 중간생략）
    MfccOptions()
        : mel_opts(23),
          // defaults the #mel-banks to 23 for the MFCC computations.
          // this seems to be common for 16khz-sampled data,
          // but for 8khz-sampled data, 15 may be better.
          num_ceps(13),
          use_energy(true),
          energy_floor(0.0),
          raw_energy(true),
          cepstral_lifter(22.0),
          htk_compat(false) {}

    void Register(OptionsItf *opts) {
    （중간생략）
    }
};
```
### defalut option (프레임 처리)

(feat/feature-window.h、struct FrameExtractionOptions)

```cpp
struct FrameExtractionOptions {
    （생략）
    FrameExtractionOptions()
        : samp_freq(16000),
          frame_shift_ms(10.0),
          frame_length_ms(25.0),
          dither(1.0),
          preemph_coeff(0.97),
          remove_dc_offset(true),
          window_type("povey"),
          round_to_power_of_two(true),
          blackman_coeff(0.42),
          snip_edges(true),
          allow_downsample(false) { }

    void Register(OptionsItf *opts) {
    （생략）
    }
   （생략）
};
```

### default option (filter bank)

(feat/mel-computations.h、struct MelBanksOptions)

```cpp
struct MelBanksOptions {
    （생략）
    explicit MelBanksOptions(int num_bins = 25)
        : num_bins(num_bins),
          low_freq(20),
          high_freq(0),
          vtln_low(100),
          vtln_high(-500),
          debug_mel(false),
          htk_mode(false) {}

    void Register(OptionsItf *opts) {
    （생략）
    }
};
```

### 파라미터로 넘겨주는 config

```cpp
--low-freq=0                 # for 16kHz sampled speech.
--high-freq=8000         # for 16kHz sampled speech.
--window-type=hamming    # 
--use-energy=false       
--num-mel-bins=24
--htk-compat=true       # try to make it compatible with HTK
--dither=0
--remove_dc_offset=false
```

kaldi에서는 디폴트로 dithering을 수행하지만, 이것을 OFF로 하면 FFT 결과까진 HTK와 같아 보인다.

그 후 filterbank처리에 대해서는 HTK에선 amplitude (power spectrum의 square root)를 사용하는 것에 비해, kaldi에선 power spectrum을 사용하기 때문에 값이 달라진다. binning이나 저주파에 대한 weighting 차이는 확인하지 않았지만, filterbank의 결과를 보면 차이가 있다.

### filterbank 처리 후(24차원)

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/ichou1/20180217/20180217100109.png)

### (참고) HCopy의 filterbank처리 후

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/ichou1/20180217/20180217100349.png)

그 후 cepstral 영역으로 옮겨 weighting하는 것은 HTK와 같다.

feature 파일을 텍스트화 하자.

```
copy-feats ark:mosi1.mfcc ark,t:mosi1_mfcc.txt
```

mosi1_mfcc.txt（1번째 프레임）

```
6.969446 -4.22486 2.13142 -5.186962 -8.526231 14.15422 6.308793 -9.442774 1.588198 -27.89196 -2.058122 -11.62059 89.52384 
```

HTK의 결과와 비교해보자. kaldi 쪽이 특징을 더 잘 식별할 수 있게 되어 있다.

![](https://cdn-ak.f.st-hatena.com/images/fotolife/i/ichou1/20180218/20180218083735.png)

--htk-compat=true 옵션은 출력 행렬의 순서를 HTK와 같도록 해준다.

> If true, put energy or C0 last and use a factor of sqrt(2) on C0.

### HTK 출력 행렬의 순서
```
1차원, 2차원, 3차원, ... 12차원, 0차원
```
### kaldi 출력 행렬의 순서
```
0차원, 1차원, 2차원, 3차원, ... 12차원
```
