## 음성인식 메모(kaldi) 2 - decode
http://work-in-progress.hatenablog.com/entry/2018/02/25/150137

---

kaldi로 실제 음성을 인식시켜보자.

아래 사이트를 참고하였다.

https://www.eleanorchodroff.com/tutorial/kaldi/kaldi-training.html

https://qiita.com/GushiSnow/items/01296c16f0d9d823ae55#_reference-726cb1a057905797dcf3

사용한 것은 '모시모시'라는 발화 데이터 (프레임 수는 198)

#### 실행 커맨드

```bash
src/gmmbin/gmm-decode-faster \
--word-symbol-table=data/lang/words.txt \
exp/tri/final.mdl \
HCLG.fst \
ark:mfcc/mosi1.ark \
ark,t:-
```

전달한 파라미터는 아래 설명에 따라

```
Decode features using GMM-based model.
Usage:  gmm-decode-faster [options] model-in fst-in features-rspecifier words-wspecifier 
Options:
--word-symbol-table         : Symbol table for words [for debug output] 
```

exp/tri/final.mdl은 triphone 모델로, HCLG.fst가 그래프.

mfcc/mosi1.ark가 feature 파일로, 결과는 텍스트 형식으로 표준출력에 쓰여지게 되어 있다.

#### Symbol table for words(data/lang/words.txt)

```
<eps> 0
!SIL 1
<UNK> 2
MOSIMOSI 3
#0 4
<s> 5
</s> 6
```

#### 결과

```
utterance_id_001 3 
utterance_id_001 MOSIMOSI 
LOG (gmm-decode-faster[5.3.106~1389-9e2d8]:main():gmm-decode-faster.cc:196) Log-like per frame for utterance utterance_id_001 is -9.01644 over 198 frames.
LOG (gmm-decode-faster[5.3.106~1389-9e2d8]:main():gmm-decode-faster.cc:209) Time taken [excluding initialization] 0.031435s: real-time factor assuming 100 frames/sec is 0.0158763
LOG (gmm-decode-faster[5.3.106~1389-9e2d8]:main():gmm-decode-faster.cc:212) Done 1 utterances, failed for 0
LOG (gmm-decode-faster[5.3.106~1389-9e2d8]:main():gmm-decode-faster.cc:214) Overall log-likelihood per frame is -9.01644 over 198 frames.
```

"MOSIMOSI"로 인식된다.

