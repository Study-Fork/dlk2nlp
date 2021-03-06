# DLK2NLP: Day-by-day Line-by-line Keras-based Korean NLP
## Sentence classification: From data construction to BiLSTM self-attention
### The tutorial is provided in both English (for *Korean* NLP learners) and Korean (for Korean *NLP* learners) - but the meanings may differ.
* 문장 분류 task를 중심으로 본 한국어 NLP 튜토리얼입니다. 본 튜토리얼에 사용된 컨텐츠 일부는 [페이스북 페이지](https://www.facebook.com/nobodybelongs/notes/)에 기고했던 글에서 발췌하였음을 명시합니다. 한국어 NLP를 하고 싶은 외국인들과 NLP를 배워보고 싶은 한국인들을 모두 대상으로 하여 영/한 설명을 모두 제공하나, 번역이 아닌 의역이며 내용도 다를 수 있습니다.


## Requirements
#### The recommended versions are in *Requirements.txt*, but can be replaced depending on the environment
fasttext, Keras, konlpy (refer to the [documentation](http://konlpy.org/en/v0.4.4/)), nltk, numpy, scikit-learn, tensorflow-gpu
#### Download [100-dimension fastText vector dictionary which was trained with 2M drama scripts](https://drive.google.com/open?id=1jHbjOcnaLourFzNuP47yGQVhBTq6Wgor) and place in the folder named *vectors*. For the utilization of the dictionary, cite the following:
```
@article{cho2018real,
	title={Real-time Automatic Word Segmentation for User-generated Text},
	author={Cho, Won Ik and Cheon, Sung Jun and Kang, Woo Hyun and Kim, Ji Won and Kim, Nam Soo},
	journal={arXiv preprint arXiv:1810.13113},
	year={2018}
}
```
#### Download the [dataset](https://github.com/warnikchow/3i4k/blob/master/data/fci.txt) and place in the folder named *data*. For the utilization of the dataset, cite the following:
```
@article{cho2018speech,
	title={Speech Intention Understanding in a Head-final Language: A Disambiguation Utilizing Intonation-dependency},
	author={Cho, Won Ik and Lee, Hyeon Seung and Yoon, Ji Won and Kim, Seok Min and Kim, Nam Soo},
	journal={arXiv preprint arXiv:1811.04231},
	year={2018}
}
```
#### For the data, an advanced version (more examples on fragments and IUs, rearrangement for some utterances) is being prepared. Might be released before 2019.
* FR과 IU를 보강하고 일부 문장들을 재배열한 advanced version의 data를 준비 중이며, 2019년이 되기 전에는 release할 계획입니다. 코퍼스 변경 시 명시해 두도록 하겠습니다.

## Contents (to be updated)
[0. Corpus labelling](https://github.com/warnikchow/dlk2nlp/blob/master/README.md#0-corpus-labeling)</br>
[1. Data preprocessing](https://github.com/warnikchow/dlk2nlp#1-data-preprocessing)</br>
[2. One-hot encoding and sentence vector](https://github.com/warnikchow/dlk2nlp#2-one-hot-encoding-and-sentence-vector)</br>
[3. TF-IDF and basic classifiers](https://github.com/warnikchow/dlk2nlp#3-tf-idf-and-basic-classifiers)</br>
[4. Dense word embeddings](https://github.com/warnikchow/dlk2nlp#4-dense-word-embeddings)</br>
[5. Document vectors and NN classifier](https://github.com/warnikchow/dlk2nlp#5-document-vectors-and-nn-classifier)</br>
[6. CNN-based sentence classification](https://github.com/warnikchow/dlk2nlp#6-cnn-based-sentence-classification)</br>
[7. RNN (BiLSTM)-based sentence classification](https://github.com/warnikchow/dlk2nlp#7-rnn-bilstm-based-sentence-classification)</br>
[8. Character embedding](https://github.com/warnikchow/dlk2nlp#8-character-embedding)</br>
[9. Concatenation of CNN and RNN layers](https://github.com/warnikchow/dlk2nlp#9-concatenation-of-cnn-and-rnn-layers)</br>
[10. BiLSTM Self-attention](https://github.com/warnikchow/dlk2nlp#10-bilstm-self-attention)

## 0. Corpus labeling
> The most annoying and confusing process. Annotation guideline should be provided to annotators, and more than two natives should be engaged in to make the labeling reliable and also for computation of inter-annotator agreement (IAA). In this project, multi-class (7) annotation of short Korean utterances is utilized.

* 데이터를 만드는, 가장 귀찮은 과정입니다. 언어학적 직관은 1인분이기 때문에, 레이블링이 설득력을 얻기 위해서는 적어도 3명 이상의 1화자를 통한 레이블링으로 그 타당성을 검증해야 합니다 (아카데믹하게는...) 
* 본 프로젝트에서는 7-class의 한국어 문장들이 분류에 사용됩니다.

---

> The task is on classification; especially about extracting intention from a single utterance with the punctuation removed, which is suggested in [3i4k](https://github.com/warnikchow/3i4k). As the description displays, the corpus was partially hand-annotated and incorporates the utterances which are generated or semi-automatically collected. Total number of utterances reaches 57K, with each label denoting</br></br>
**0: Fragments**</br>
**1: Statement**</br>
**2: Question**</br>
**3: Command**</br>
**4: Rhetorical question**</br>
**5: Rhetorical command**</br>
**6: Intonation-dependent utterance**</br></br>
where the [inter-annotator (IAA)](https://en.wikipedia.org/wiki/Cohen%27s_kappa) was computed 0.85 (quite high!) for the manually annotated 2K utterance set (corpus 1 in the table below). The [annotation guideline](https://drive.google.com/open?id=1AvxzEHr7wccMw7LYh0J3Xbx5GLFfcvMW) is in Korean.</br>
<p align="center">
    <image src="https://github.com/warnikchow/3i4k/blob/master/images/portion.PNG" width="400">
    
* 태스크는 의도 분류로써, [3i4k](https://github.com/warnikchow/3i4k) 프로젝트를 위해 제작된 DB를 사용합니다. 사실 국책과제에 쓰려고 만든건데 어차피 논문으로도 submit했으니 공개는 상관 없지 않을까 싶어요. 5만 7천 문장쯤으로 아주 규모가 크지는 않지만, 일단 수작업으로 2만 문장 정도에서 0.85의 IAA를 얻었으며 (꽤 높은 agreement!), 4만 문장 가량이 더 수집/생성되어 그래도 어느정도 쓸만한 데이터셋이 만들어졌습니다. 

* 레이블 7개는 위에 써 둔 것처럼, Statement~Rhetorical question까지의 clear한 의도 5가지와 (논문에선 clear-cut cases라고 칭했습니다만), 의도가 불분명한 Fragment (명사, 명사구, 혹은 불완전한 문장), 마지막으로 Intonation-dependent utterances *억양에 따라 의도가 달라지는 문형* 입니다. 마지막 레이블은 저 논문에서 하나의 레이블로 하기로 제안한 것이지만, 한국어 화자라면 어떤 문장들이 그런 성질을 가지는지 감이 올 것입니다. "뭐 먹고 싶어" "천천히 가고 있어" 같은 문장들이 그러한 유형이죠. Spoken language understanding에 아주 골머리를 썩이는 녀석들이기 때문에 따로 분류하기로 하였습니다. 

* Annotation guideline이 어떤 형식인지 궁금하신 분들은 [이곳](https://drive.google.com/open?id=1AvxzEHr7wccMw7LYh0J3Xbx5GLFfcvMW)을 참고하시면 됩니다.

## 1. Data preprocessing
> For the next step, we should pay attention to how we can manage HANGUL, the letters of Korean writing system (WS). Korean WS, letter and its alphabets *Jamo* which was (maybe solely) invented by The Great King Sejong, incorporate special morpho-syllabic blocks which have a role of syllable and consist of CV(C), making Korean as a representative language with featural WS. The blocks are agglutinated to make up the word *eojeol*, which should be decomposed into morphemes for a semantically meaningful language processing. The spacings go between the *eojeol*s, to enhance the readability of a sentence. Many morphological analyzers give an additional spacing between the morphemes; some analyzers such as [Twitter](https://github.com/twitter/twitter-korean-text) simply give spaces, and there are the ones which conduct an elaborate tokenizing process of block decomposition, such as [Kkma](http://kkma.snu.ac.kr/). Total five analyzers are wrapped in the famous Korean natural language processing toolkit, [KoNLPy](http://konlpy.org/en/v0.4.4/). In this tutorial, we proceed with the Twitter analyzer, for its speed and to prevent the letters being decomposed.

* 한국어와 한글, 문자 체계, 형태소, 띄어쓰기 및 형태소 분석기에 대해 주절주절 설명해 봤는데요, 아마 한국어 L1 화자라면 대부분 익숙한 내용이실 테니 이 부분의 한글 설명은 생략하도록 하겠습니다 ㅎㅎ 

* 아직 국제 무대에서는 크게 주목받지는 못하지만 그래도 한국어는 화자 수로만 봐도 세계 15위권 안에 들고 진짜로 몇 안되는 featural writing system (갓종갓왕님 1인 프로젝트 덕분에 sign writing이나 소설을 위해 만들어진 언어 등과 같은 문자형식 지위를 획득 ...) 이며 promissive와 같은 독특한 particle 덕분에 언어학 서적에서도 왕왕 나오는 언어 및 문자체계를 갖고 있습니다. 또한 BTS의 떡상으로 전세계 사람들이 한국과 한국어와 한글을 알아서 Korean NLP가 국제무대에서도 Chinese나 Arabic만큼 중요하게 다뤄지는 때가 오기를... 혹은 통일을 기원하며 떡상을 기다리는... 

* 여튼 여기서 한 얘기는 한국어 NLP에서 semantic하게 의미있는 작업을 위해서는 agglutinate된 block들을 morpheme 단위로 적절히 쪼개는 과정이 필수적이라는 얘기를 하고 있습니다. 저는 KoNLPy에 들어 있는 Twitter가 가볍고 자소분해를 하지 않아 많이 사용하는데, 여기서도 해당 모듈을 사용하도록 하겠습니다.

---

> The most important part of Korean NLP lies in using Python 3.x. For the lower version, the encoding issue will bother you. That is, in Python 3.x, you don't have to perform an additional encoding for HANGUL to be read and written. [THIS](https://github.com/warnikchow/3i4k/blob/master/data/fci.txt) is the file URL, which is a single *.txt* file that has the first column of the labels and the second column of the sentences. The dataset is split into train and test set of ratio 9:1 and the test set denotes last 10% of the corpus. The following code reads the dataset into the sentences and the labels, where the dataset is placed in the path 'data/fci.txt' for the directory you're running the console.

```python
def read_data(filename):
    with open(filename, 'r') as f:
        data = [line.split('\t') for line in f.read().splitlines()]
    return data
    
fci = read_data('data/fci.txt')
fci_data = [t[1] for t in fci]
fci_label= [int(t[0]) for t in fci]
```

* 한국어엔 역시 Python 3.x 죠.... 2.x로 한국어 인코딩은 지옥입니다 ㅠㅠ 처음에 아주 고생했네요. 어쨌든 제일 첫 삽은 [data](https://github.com/warnikchow/3i4k/blob/master/data/fci.txt)를 로딩하고 sentence와 label들로 나누는 과정입니다. 저는 [Lucy Park님의 유명한 슬라이드](https://www.lucypark.kr/docs/2015-pyconkr.pdf)에서 처음 봤지만, 외국 NLP 튜토리얼들에도 비슷하게 나와 있습니다. 일단 data라는 폴더를 만들어서 *fci.txt* 파일을 넣어주셔야 하며, read_data로 파일을 읽어들이고 탭(\t)으로 split한 후, data와 label array로 나누게 됩니다.

---

> The last part of data preprocessing is tokenizing the sentence into morphemes, as emphasized previously. Although many character-based (morpho-syllabic blocks) or alphabet-based (consonants and vowels, or *Jamo*) approaches are utilized these days, the morpheme-based approach is still meaningful due to the nature of Korean as an agglutinative language. For the sparse vector classification such as one-hot encoding and TF-IDF which will be displayed in the following chapter, we will adopt the morpheme sequence which can be obtained by the Twitter tokenizer.

```python
import nltk
from konlpy.tag import Twitter
pos_tagger = Twitter()

def twit_token(doc):
    x = [t[0] for t in pos_tagger.pos(doc)]
    return ' '.join(x)

len_train = int(np.floor(len(fci_data)*0.9))
fci_data_train  = fci_data[:len_train]
fci_data_test   = fci_data[len_train:]
fci_label_train = fci_label[:len_train]
fci_label_test  = fci_label[len_train:]

fci_token_train = [twit_token(row) for row in fci_data_train]
fci_token_test  = [twit_token(row) for row in fci_data_test]

fci_sp_token_train = [nltk.word_tokenize(row) for row in fci_token_train]
fci_sp_token_test  = [nltk.word_tokenize(row) for row in fci_token_test]
```

* 데이터를 읽어들였으니, 이제 형태소 분리를 통해 어절이라는 큰 뭉텅이들을 좀 더 세밀한 단위로 나눠 보도록 하겠습니다. 음절 기반 방법, 혹은 자소 분해 방법들도 요즘 많이 사용되지만, 형태소 기반의 문장 표현이 아무래도 교착어인 한국어에서 가장 기본적인 접근 방법이 아닌가 싶습니다. 

* 아까 말씀드렸듯 트위터 형태소 분석기를 사용하여 어절들을 분리할 계획이며, 이는 one-hot encoding이나 TF-IDF를 이용한 sparse vector classification에 활용하기 위함입니다.

## 2. One-hot encoding and sentence vector
> The fundamental of computational linguistics lies in making machines understand human utterances (or, natural language). Since it is difficult for even human beings to understand the **real meaning**, the first step is to represent the words and sentences into the numerics that are computable for the machine learning systems. Given the dictionary of vocabulary size N, the most famous and intuitive approach is **one-hot encoding** that assigns 1 for only one entry if a specific word is given.

* 전산 언어학은 기본적으로 컴퓨터한테 사람이 말하는 것, 즉 자연어를 이해시키는 과정이라고 할 수 있겠습니다. 그래서 우리는 컴퓨터로 하여금, '아 진짜 우리 말을 이해하진 못해도, 대충 어떤 내용인지 판단은 할 수 있게 해 보자' 생각을 하게 됩니다. 그렇게 해서 나오게 된 표현법 (representation) 중 가장 기본적으로 사용되고, 매우 강력한 편이라 아직까지도 많은 곳에서 사용하고 있는 방법론은 바로 단어의 one-hot encoding입니다. 

* 이 방법을 쓰기 위해선 '우리가 생각할 단어 전체 set = Dict'을 생각해야 합니다. 가장 먼저 생각해볼 수 있는 건 우리가 사용할 코퍼스에 있는 모든 단어들의 모임이죠. 코퍼스의 사이즈가 커질수록 Dict도 커질 것이고, 물론 단어는 countably infinite하게 만들어낼 수 있지만 여기선 '그나마 자주 쓰이는 녀석들'로 생각합시다. 

* 이 Dict가 생각보다 커서 실제로는 보통 많이 쓰이는 n만 단어 같이 제한을 둬서 잡습니다. 목적에 따라 functional한 녀석들은 아예 카운트하지 않기도 해요. 이 때 size(Dict) = V 이라 하면, 우리는 (고려하기로 한) 모든 단어들을 V-dim의 one-hot vector로 표현할 수 있게 되는 겁니다.

---

> What should we do with these large-dimensional vectors? The first thing we can think of is a feature called **Bag-of-Words** (BoW). The literal meaning is a bag which contains words, and for Korean, it might be either words (*eojeol*), morphemes, characters, or alphabets (*Jamo*). Though we've decided only to use morphemes for the sparse representations, these can be replaced with whatever feature you want to adopt. BoW approach is straightforward; it just assigns 1 to the corresponding entry if the word in the sentence. This can provide the computer with a numerical value which represents the sentence! The code utilizing [NLTK](https://github.com/nltk/nltk) is under construction.

* 이걸 갖고 뭘 하느냐? 가장 먼저 생각해볼 수 있는 것은 bag-of-words란 녀석입니다. 말 그대로 '단어가 든 가방' 이에요. 이 때 가방 = 문장 입니다. 문장 안에 어떤 단어들이 들었냐를 V-dim one-hot vector들의 OR-sum operation (하나만 있어도 1됨) 으로 표현하는 거죠. 

* 예컨대 "신이 그댈 사랑해"라는 문장이 있고 '신이' = \[1 0 0 0 0 0 \], '그댈' = \[0 0 1 0 0 0\], '사랑해' = \[0 0 0 0 0 1\]로 표현된다고 합시다. 여기서 코퍼스는 여섯 개의 토큰 (단어구성단위 라고 합시다 일단)으로 구성된 아주 작은 Dict를 yield했겠지요. 그렇다면 상기 문장은 \[1 0 1 0 0 1\]의 size(Dict)-dim binary vector로 표현되는 겁니다. 이렇게 해서 뭘 할 수 있냐구요? 이제 컴퓨터도 알아먹는 수치적 정보가 되었으니, 각종 분류기에 넣어 재미를 볼수 있죠! 

* 물론 태클이 들어올 수 있습니다. 저 문장을 사실 '신' '이' 그대' '-ㄹ' '사랑' 'ㅎ' '-애' 로 나눠야 합당하지 않느냐, 한 문장에서 여러 번 카운트되는 단어들이 있으면 one-hot은 부당한 representation이 아니냐 뭐 그런... 첫 번째의 경우 우리가 문장을 자소로 분리하지 않는 Twitter analyzer을 썼기 때문에 어쩔 수 없는 부분입니다. 두 번째의 경우 다음 chapter에서 더 다뤄 보도록 하겠습니다.

## 3. TF-IDF and basic classifiers
> Previously, we've introduced one-hot encoding of the words and the sparse sentence representation based on the BoW model. However, despite its transparency and conciseness, one-hot encoding does not convey the word frequency regarding the document. This is where the concept of **term frequency** (TF) came out; the word frequency is taken into account to convey the relative importance of each word. For instance, the word 'I' and 'you' in the sentence *I love you, I want you, I need you* may be assigned the word frequency of 3 instead of 1 which is assigned to the verbs.

<p align="center">
    <image src="https://github.com/warnikchow/dlk2nlp/blob/master/image/tfidf.png" width="400"><br/>
    (Image from https://skymind.ai/wiki/bagofwords-tf-idf)

* 앞서 bag-of-words 모델을 통해 문장을 1-hot vector들의 합(여기서 합은 Boolean의 or에 해당)으로 나타내어 컴퓨터가 알아먹을 만한 어떤 수치로 나타내는 과정을 설명했습니다. 이는 전통적이고 직관적이며 상당히 강력한 방법이기도 합니다.

* 하지만 이 방법의 문제는 문장 내에 등장하는 모든 단어들을, 등장 횟수에 상관없이 공평하게 대한다는 것입니다. 예컨대 "나는 아브라함의 하나님이요 이삭의 하나님이요 야곱의 하나님이로라 하신 것을 읽어 보지 못하였느냐 (전 종교가 없습니다! 그냥 예문을 찾고싶었을뿐)" 라는 문장이 하고 싶은 말은 누가 봐도 내가 하나님이란 것 같은데 아브라함이니 이삭이니 하는 것들과 동급으로 1로 카운트되면 얼마나 억울하겠습니까. 

* 이러한 문제를 해결하기 위해 우린 일종의 normalization으로 볼 수 있는 term frequency, 즉 문장의 전체 word중 해당 word의 빈도를 고려해 문장을 벡터화해 볼 수 있습니다. 위의 문장에선 '하나님'이 3/X (형태소분석한 결과를 모두 카운트하기 귀찮으니 X로 퉁칩시다)의 값을 부여받고 나머지 녀석들은 1/X의 값을 부여받게 되겠죠.

---

> However, the problem is that the high frequency does not indicate that importance of the word. Thus, the concept of **inverse-document frequency** (IDF) is introduced, as a way of multiplying the inverse fraction of the frequency of the word among the whole documents. This prevents the overestimation of the functional words in many cases; to be honest, 'I' and 'you' are not as important as 'love', 'want' and 'need'. For instance, in the morpheme-based analysis, many particles in Korean are used repetitively in the sentences; those will be assigned a low IDF so that the lexical words are emphasized alternatively. The following code utilizing the scikit-learn library demonstrates how the **term frequency-inverse document frequency** (TF-IDF) is computed for our corpus.

```python
# Referred to the followings for the code:
# https://gist.github.com/jason-riddle/1a854af26562c0cdb1e6ff550d1bf32d#file-complex-tf-idf-example-py-L40
# http://blog.christianperone.com/2011/10/machine-learning-text-feature-extraction-tf-idf-part-ii/

from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfTransformer
from sklearn.feature_extraction.text import TfidfVectorizer

def tfidf_featurizer(corpus_train,corpus_test):
    count_vectorizer = CountVectorizer()
    count_vectorizer.fit_transform(corpus_train)
    freq_term_matrix = count_vectorizer.transform(corpus_train)
    tfidf = TfidfTransformer(norm="l2")
    tfidf.fit(freq_term_matrix)
    # unigram features
    tfidf_token = TfidfVectorizer(ngram_range=(1,1),max_features=3000)
    token_tfidf_total = tfidf_token.fit_transform(corpus_train+corpus_test)
    token_tfidf_total_mat = token_tfidf_total.toarray()
    # bigram features
    tfidf_bi_token = TfidfVectorizer(ngram_range=(1,2),max_features=3000)
    token_tfidf_bi_total = tfidf_bi_token.fit_transform(corpus_train+corpus_test)
    token_tfidf_bi_total_mat = token_tfidf_bi_total.toarray()
    return token_tfidf_total_mat,token_tfidf_bi_total_mat

fci_tfidf,fci_tfidf_bi  = tfidf_featurizer(fci_token_train,fci_token_test)
fci_tfidf_train    = fci_tfidf[:len_train]
fci_tfidf_test     = fci_tfidf[len_train:]
fci_tfidf_bi_train = fci_tfidf_bi[:len_train]
fci_tfidf_bi_test  = fci_tfidf_bi[len_train:]
```

* 그런데 또 문제가 있습니다. 바로 하나님이 소유를 나타내는 '의'와 동급이 돼버리는 겁니다. 아아, 이렇게 원통할 데가... 딱 봐도 '의'라는 녀석은 문장의 의미를 판단하는 데에 큰 도움을 주지 않을 것으로 보입니다. 다른 문장들에도 많이 나올 게 분명하거든요. 

* 그래서 우리가 생각해볼 수 있는 건 '다른 문장들에도 많이 나오는 녀석엔 가중치를 조금 주면 어떨까?'하는 것입니다. 예컨대 전체 문장 중 해당 문장이 나오는 비율의 역수 같은 걸 곱해준다면? 이런 생각으로 나온 녀석이 바로 inverse document frequency입니다. 약간의 smoothing factor을 추가하자면, test corpus에 해당 term이 없는 경우를 대비해 분모에 1을 더해주고, corpus size가 방대해질 때를 고려해 log를 입히는 정도?

* 상기한 두 개의 요소를 곱해 BoW를 개량한 모델이 바로 TF-IDF (term frequency-inverse document frequency) 입니다. 문서 분류에 아직도 활발히 사용되는 모델이지요. 

---
> The aforementioned sentence representation can be directly utilized with the basic classifiers such as naive Bayes (NB), decision tree (DT), support vector machine (SVM) and logistic regression (LR). The evaluation is done with accuracy and F1-score; the accuracy refers to the fraction of correct instances out of the whole data. Understanding the meaning of the value is intuitive, but the flaw is that it does not convey how incorrect the model predicts for the class of data with a small portion. For example, if the data consists of the binary label and the portion is 9:1 yielding an imbalance, then the accuracy may reach 90% just by a simple guess of a class with the larger volume. The better solution can be obtained by using F1-score, which considers the true negatives and the false positives. The following code displays how the sparse vectors we previously obtained are trained and predicted, with both an accuracy and an F1-score.

```python
from sklearn.linear_model import LogisticRegression
from sklearn import metrics
from sklearn.metrics import accuracy_score, precision_recall_fscore_support

classifier_uni = LogisticRegression(random_state=1234)
classifier_uni.fit(fci_tfidf_train,fci_label_train)
uni_pred = classifier_uni.predict(fci_tfidf_test)
accuracy_score(uni_pred,fci_label_test)
metrics.f1_score(uni_pred,fci_label_test,average="macro")
metrics.f1_score(uni_pred,fci_label_test,average="weighted")
precision_recall_fscore_support(uni_pred,fci_label_test)

classifier_bi = LogisticRegression(random_state=1234)
classifier_bi.fit(fci_tfidf_bi_train,fci_label_train)
bi_pred = classifier_bi.predict(fci_tfidf_test)
accuracy_score(bi_pred,fci_label_test)
metrics.f1_score(bi_pred,fci_label_test,average="macro")
metrics.f1_score(bi_pred,fci_label_test,average="weighted")
precision_recall_fscore_support(bi_pred,fci_label_test)
```

* 지금까지  tf-idf를 이용한 sentence 의 sparse representation에 대해 얘기했지요. 이제 구체적으로 이 녀석들을 문장 분류에 이용해먹을 만한 방법들에 대해 생각해봐야 하는데요, 그 중 하나가 이 글의 task로 제시된 분류 (classification)입니다. 

* 사실 데이터 집합(문장들)과 레이블 집합(긍정문/부정문, 의문문별 분류, 토픽별 분류 등)으로 된 트레이닝 데이터를 input으로 넣어, 별도의 테스트 데이터로 얻어진 prediction와 정답의 비교를 통해 그 네트워크의 accuracy, F-measure 등을 evaluate하여 성능을 평가한다는 점은 많은 분야에서 사용되는 분류기 evaluation의 구조입니다. 

* 이 때 accuracy는 전체 테스트 인풋 중 분류에 성공한 input의 비율이 될 겁니다. 그런데 함정이 있다면, 데이터 편중을 배제하기 어렵단 것이죠. 예컨대 데이터셋의 90퍼센트가 긍정문이고 10퍼센트만 부정문이라면, 공정한 트레이닝 및 테스트가 되었다고 하기 어렵겠죠. 그래서 binary classification에서 많이 사용되는 F-measure의 경우는 precision과 recall이라는, 각각 '1이라고 했을 때 정말 맞을 확률'과 '전체 맞은 것들 중 1이라고 했을 확률'을 생각하여 이들의 조화평균을 구해주는 방향으로 accuracy의 함정을 보정합니다. 물론 단순 조화평균이 아닌 별도 parameter을 주기도 하고, multiclass에 대해선 따로 class별 weight를 정의하기도 하지만요.

---

> The result is as below; it is weird that the bigram features yielded quite a lower accuracy. It is assumed that the property of the corpus, where the sentence label does not depend solely on the polarity items or emotion words, may have affected the performance. Also, the dimension of the feature (3,000) which was fixed to guarantee the fair comparison between the two models, could have been a harsh obstacle for the bigram model.

```properties
# CONSOLE RESULT
>>> accuracy_score(uni_pred,fci_label_test)
0.7785129723141215
>>> precision_recall_fscore_support(uni_pred,fci_label_test)
(array([0.67030568, 0.89564732, 0.87613122, 0.75669291, 0.13414634,
       0.21153846, 0.02673797]), array([0.69300226, 0.70798412, 0.86584684, 0.82277397, 0.61111111,
       0.70967742, 0.55555556]), array([0.68146504, 0.79083518, 0.87095867, 0.78835111, 0.22      ,
       0.32592593, 0.05102041]), array([ 443, 2267, 1789, 1168,   36,   31,    9]))
>>> metrics.f1_score(uni_pred,fci_label_test,average="macro")
0.5326509049311736
>>> metrics.f1_score(uni_pred,fci_label_test,average="weighted")
0.7996055049065471

>>> accuracy_score(bi_pred,fci_label_test)
0.3668814208601776
>>> precision_recall_fscore_support(bi_pred,fci_label_test)
(array([0.75327511, 0.48158482, 0.36029412, 0.20551181, 0.        ,
       0.        , 0.00534759]), array([0.2457265 , 0.37850877, 0.53983051, 0.30278422, 0.        ,
       0.        , 1.        ]), array([0.37056928, 0.42387033, 0.43215739, 0.24484053, 0.        ,
       0.        , 0.0106383 ]), array([1404, 2280, 1180,  862,    5,   11,    1]))
>>> metrics.f1_score(bi_pred,fci_label_test,average="macro")
0.21172511891093732
>>> metrics.f1_score(bi_pred,fci_label_test,average="weighted")
0.38441799201505655
```

* 결과는 뜻밖이었는데요, bigram으로 구한 결과가 그다지 신통치 않은 것 같습니다. (코드를 잘못 짰나...?) 어쨌든 궁색하게 분석을 해보자면, 본 task가 sentiment analysis 처럼 단어 한두개로 문장의 의미가 크게 달라지는 task들과 다르게 전체적인 맥락 및 어순을 고려해야 하기도 하고, fair comparison을 위해 3,000으로 제한한 feature dimension이 오히려 bigram에는 해가 되어 정작 중요한 형태소들을 놓친 게 아닌가...하는 생각이 듭니다.

* 앞서 구한 one-hot encoding으로도 이러한 evaluation을 손쉽게 할 수 있습니다. 방금 TF-IDF에서 그랬듯, evaluation을 위한 prediction은 10%의 test data로 하게 되지요. Bigram TF-IDF는 좀 불만족스럽지만, 어쨌든 sparse representation으로도 어느 정도는 높은 정확도를 가진 모델을 얻을 수 있음을 알 수 있습니다. 물론 일부 class들에 대해선 random guess만도 못한 결과를 내고 있지만요. 다만 아직도 찝찝한 점은, 컴퓨터가 단어를 세고만 있지 그 단어들이 문장 내에서, 작게는 컨텍스트에서 어떤 역할을 하는지 아무것도 모르는 것 같다는 점입니다. 

## 4. Dense word embeddings

> Three-line summary:</br>
> 1. Computational linguistics aims making machines understand human language.</br>
> 2. As a fundamental approach for the representation of word and sentence, one-hot encoding and TF-IDF are introduced.</br>
> 3. For Korean, due to the property of agglutinative language, morpheme-based analysis can be more effective than the word (*eojeol*)-based one.</br>

> However, considering the computation issue which has been very important up to this date, the sparse representations may not be the optimal solution for the contemporary neural network-based systems. This is the point where the dense representation such as [*Word2Vec*](https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf) is successfully adopted.

* 여태까지 한 내용을 세줄요약해 보면 다음과 같습니다.</br>
1. 전산언어학은 기계로 하여금 사람 말을 잘 알아먹도록 하는 학문</br>
2. 단어 및 문장을 모델링하는 기본적인 방법으로 one-hot encoding (BoW) 및 TF-IDF가 있음</br>
3. 한국어는 교착어적 특징 때문에 어절 단위 분석보다 형태소 단위 분석이 효과적일 수 있음</br>

이외에도 수치화된 문장을 분류하는 알고리즘에는 SVM 이나 LR 등이 있다, 그 성능을 평가하는 measure에는 accuracy나 F1 score 등이 있다 ... 라는 것도 간략히 말하고 지나왔죠. 

* 그런데 이러한 방법론들은 분석해야 할 데이터가 많아지고 사전의 크기가 커질수록 computation의 문제에 당면하게 됩니다. 특히나 데이터를 몰빵해넣고 병렬연산으로 승부하는 딥러닝의 경우 더욱 그렇지요. 예컨대 30 개의 형태소로 된 문장을 벡터 sequence로 나타내어 recurrent neural network 같은 시스템으로 요약하고자 할 때, 딕셔너리 사이즈가 10만이라면, 10만 x 30이라는 무시무시한 사이즈의 행렬이 문장 하나를 표현하여 인풋으로 들어가게 되는거죠. 물론 아까 보았듯 사용하는 벡터의 크기는 조절할 수 있고, 각 벡터가 각 단어를 표현하는 것이 아주 명확하긴 하지만요.

* 이럴 때 유용하게 사용되는 개념이 2013년 태동한 word2vec, 혹은 dense low-dimension embedding 입니다. 등장 목적이 위와 같다고는 할 수 없지만, [2006년부터 촉발된 딥러닝 아키텍쳐](http://www.cs.toronto.edu/~fritz/absps/ncfast.pdf)와 맞물려 사용되며 문장 분석의 패러다임 자체를 바꾸고 있죠. Word2vec, GloVe, fastText에 대한 설명을 간략하게만이라도 해보려 합니다.

---

> The term word2Vec is very intuitive, and it converts the words, which are discrete (and were sparsely represented so far), into the numerics that are close to the continuousness. Since there are a bunch of terrific articles on the topic outside, in English, a review on word2vec and its related models are discussed mainly in Korean.

<p align="center">
    <image src="https://github.com/warnikchow/dlk2nlp/blob/master/image/skipgram.png" width="400"><br/>
    (Image from [the tutorial site](https://sheffieldnlp.github.io/com4513-6513/lectures_reveal_js/lecture12_word_embeddings.html). The matrix W x D is obtained by minimizing the cost function regarding the performance of context prediction given an input vector.)

* word2vec이라는 말은 상당히 직관적입니다. 말 그대로 이산적 개념인 단어를 수치적 개념인 벡터로 바꿔 준다는 의미이죠. 사실 one-hot encoding 역시 고차원의 벡터를 만들어준다는 점을 생각하면 어폐가 있긴 합니다. 그래서 두 개념의 차이를 벡터가 sparse한지 (드문드문하게 nonzero인 성분이 있는지), 아니면 dense한지 (유클리드 공간에서처럼 빽빽이 들어차 있는지)를 차이점으로 봅니다. 물론 word2vec은 후자를 의미하죠.

* 당연한 얘기겠지만, 어떤 방식의 워드 임베딩이든, 임베딩 벡터 셋을 트레이닝하는 과정에서 어떤 원칙 (혹은 제약조건)을 주냐에 따라 결과물로 나오는 벡터들 간의 관계가 달라지게 됩니다. 예컨대 아무 제약 조건도 주지 않는 one-hot encoding의 경우, 모든 단어가 평등합니다. 아무리 비슷해 보이는 단어들이라도 1이 위치하는 엔트리가 다르다면 아무 상관없는 단어인 것이나 다름없게 되는 것이죠. distributional semantic에 대한 고려는 one-hot vector에 들어 있지 않은 겁니다.

* 모든 sparse vector가 one-hot vector같이 엔트리 간 equivalence를 지니는 건 아닙니다. 오히려 희소한 케이스에 가깝죠. 가중치를 곱해준 TF-IDF만 해도, 개별 단어를 모델링하지는 않지만 어떤 분포적인 특성이 반영되게 됩니다. binary이지만 multi-hot encoding을 차용하는 [boolean distributional semantics](https://transacl.org/ojs/index.php/tacl/article/view/616)의 경우, 의미론에서 얘기하는 feature의 개념이 등장하여, taxonomy 상 상위 위계에 해당하는 단어의 embedding이 하위 위계의 단어의 embedding에 포함되게 하는 방식으로 트레이닝이 됩니다. 아마도 벡터 사이즈는 one-hot보다는 줄어들겠죠? 벡터들 간의 distance measure가 euclidean이 아닌 어떤 이산적인 개념이 된다는 건 challenging하긴 합니다만, 언어가 기본적으로 이산적인 속성을 버릴 수 없다는 생각을 갖고 있는 저로썬 매우 매력적이라는 생각을 했습니다.

* 그렇지만 이상은 이상이고, 우리는 많은 순간 현실과 타협해야 합니다. 작은 벡터에 정보들을 우겨 넣어야 하죠.  또한 back propagation과 같은 연산을 통해 이루어지는 최신 최적화 기법들의 수혜를 받으려면, 벡터 간의 거리가 어떤 미분가능한, 이산적이지 않은 개념으로 정의가 되어야 함도 사실입니다 (물론 이산적인 목적 함수들을 위해 별도의 신경망을 연결해 주는 알고리즘도 있는 것으로 압니다만 일단 그건 나중에 생각하도록 하지요). 그러기 위해서, 고차원의 벡터를 저차원에 임베딩해 넣으면서도, 단어들 간의 관계가 좀 더 유기적으로 연결될 수 있는 방법엔 뭐가 있을까요? 

* 이러한 관점에서 볼 때, word2vec은 상당히 매력적인 시도였다 볼 수 있겠습니다. 혹자는 '비슷한 context에 등장하는 녀석들은 실제로도 비슷한/관련 있는 녀석들일 가능성이 높다는 distributional semantics의 원칙을 수치적 제약으로 잘 지키면서 one-hot vector을 저차원에 pca한 결과물'이라고 하더군요. 예컨대 '너는 나쁜 아이야'와 '너는 착한 아이야'라는 문장들을 볼 때, '나쁜'과 '착한'이 실제로 저런 류의 context를 많이 공유한다 생각해 보면, 둘이 아주 관계가 없는 단어들은 아니다, 어느정도 유기적이다, 그런 판단을 할 수 있겠죠? word2vec의 큰 철학은 이렇습니다. 컨텍스트를 주고 center word를 추론하는 CBOW와 그 반대인 skip-gram (SG) 모두 word2vec의 최적화와 관련된 알고리듬인데요, SG가 여러 태스크에서 더 성능이 좋음이 보여진 바 있고 실제로도 더 자주 활용됩니다. 

---

> Up to date, many advanced models of word vectors which base on word2vec have been proposed. For instance, [**GloVe**](https://nlp.stanford.edu/pubs/glove.pdf) represents the word vectors considering the co-occurrence in the window, and [**fastText**](https://arxiv.org/abs/1607.04606) utilizes the subword n-gram so that the embedding can be efficient for morphologically rich languages. In the following approaches that use dense word vectors, we will adopt [100-dimension fastText vector dictionary which was trained with 2M drama scripts](https://drive.google.com/open?id=1jHbjOcnaLourFzNuP47yGQVhBTq6Wgor), for the project 3i4k.

```python
import fasttext
model_ft = fasttext.load_model('vectors/model_drama.bin')
```

* Word2vec은 이런 저차원 임베딩의 포문을 열었을 뿐이고 (for문 아닙니다) 이후로 수많은 베리에이션이 등장했습니다. 이 때 사용하는 알고리듬이 skip-gram이란 점이 크게 변하지는 않았지만, 다양한 objective를 설정하며 특정 task에 효율적으로 적용할 수 있는 vector들이 등장했죠.

* 가장 많이 알려진 두 베리에이션은 GloVe와 fastText입니다. 전자는 word2vec 이전에 사용되던 sparse representation 중 하나인 co-occurrence matrix의 개념을 training objective에 활용하여 보다 global하고 syntactic한 특징도 word vector에 반영할 수 있게 한 것이고, 후자는 word2vec과 사실상 같은 알고리즘이지만 그 트레이닝 효율을 거의 몇백배 수준으로 끌어올려 빠르게 트레이닝하면서도 task 성능은 엇비슷하게 유지할 수 있게 하는 알고리듬입니다. 또한 fastText는 subword n-gram model을 차용하여, word 단위로 나뉘어지지 않은, 단어 내의 character n-gram들도 일종의 word로 보고 그 분포를 전체 트레이닝에 고려한다는 특징을 가지고 있지요. 이를 통해 morphologically rich한 언어들에서도 효율적으로 활용 가능하다는 점을 논문에서 어필하고 있구요.

* GloVe의 경우 stanford에서 제공하는 [wiki/twitter기반 pre-trained vector](https://nlp.stanford.edu/projects/glove/)가 있으며, fastText의 경우는 꽤 많은 언어로 pre-trained vector을 제공하지만 학습 속도가 굉장히 빨라 저 같은 경우는 갖고 있는 코퍼스로 새로 training하기도 합니다. Subword n-gram이 교착어인 한국어에도 굉장히 유용하여, 저 같은 경우는 fastText로 트레이닝된 word vector set을 character embedding에 사용하고 있구요. 약 200만 문장의 드라마 스크립트를 통해 학습한 word vector dictionary는 [다음의 주소](https://drive.google.com/open?id=1jHbjOcnaLourFzNuP47yGQVhBTq6Wgor)에서 제공됩니다.

* 앞서도 말했지만, 26개의 알파벳만 있으면 되는 영어와 달리, 한국어는 2500개 상당의 자모 조합이 있어 one-hot encoding을 직접적으로 character embedding에 사용하기 쉽지 않죠. 이럴 때 유용하게 사용할 수 있는 것이 저차원으로 임베딩된 character vector입니다. 어느 정도 분포적인 특징을 반영할 수 있으면서도 computational하게 부담을 덜 줄 수 있는 그런 feature로 사용할 수 있는 것입니다. 물론 형태소, 어절 모두 임베딩의 대상이 될 수 있습니다. 어떤 것을 선택할지는 형태소 분석기의 유무, 구동 및 개발 환경 등에 따라 자유롭게 선택하면 될 것입니다. 

## 5. Document vectors and NN classifier

> In the first few chapters, we've demonstrated how the corpus we've adopted is preprocessed, featurized, trained and predicted with the sparse sentence encodings. However, since we've obtained the dense word embeddings for the morphemes (as we've obtained for one-hot encoded words), it's plausible to extend it to the sentence vector, for instance by summation. 

```python
import numpy as np
from numpy import linalg as la

def featurize_nn(corpus,wdim):
    nn_total = np.zeros((len(corpus),wdim))
    for i in range(len(corpus)):
        if i%1000 ==0:
            print(i)
        s = corpus[i]
        count_word=0
        for j in range(len(s)):
            if s[j] in model_ft:
                nn_total[i,:] = nn_total[i,:] + model_ft[s[j]]
                count_word = count_word + 1
        if count_word > 0:
            nn_total[i] = nn_total[i]/la.norm(nn_total[i])
    return nn_total

fci_nn = featurize_nn(fci_sp_token_train+fci_sp_token_test,100)
```

* 앞서 one-hot encoding과 tf-idf로 문장을 수치화하는 방법, 그리고 그것이 sparse vector이라는 것까지 말씀드린 바 있습니다. 정보가 discrete하다는 것은 기계에게나 사람에게나 매우 직관적으로 다가오기에, 문장의 분류에 있어 굉장히 유용하게 사용된다는 것두요. 하지만 지난 몇 시간 동안 dense word vector들을 얘기하는 동안, 그 친구들을 문장 수치화에 이용할 수 있지 않을까 생각하는 분도 분명 계실 겁니다. word2vec을 제안한 사람들의 머릿속을 들여다볼 수는 없지만, 결과적으로 그 등장이 문장 임베딩에 있어 하나의 패러다임 전환이 되었죠.  

* “I really love you” 이라는 영어 문장을 한번 생각해 봅시다. 일단 가장 먼저 생각해볼 수 있는 것은, \[0 0 0 ... 1 ... 1 ... 1 ... 1 ... 0 0 0\] 이런 식으로 나타내어진 multi-hot vector (one-hot vectors의 or summation)이겠죠. 그 다음은 \[0 0 0 ... 0.3 ... 0.7 ... 0.9 ... 0.4 ... 0 0 0\] 이런 식으로 표현된 tf-idf일 겁니다. 이제 word2vec이 무엇인지 배웠으므로, 이런 방법도 생각해 볼 수 있을 겁니다. ‘문장 내 모든 word와 word embedding function f에 대해, f(w)를 모두 더하기’ 즉, 워드 임베딩이 100차원의 real vector이라면, 전체 sentence vector s=sum(f(w))도 100차원의 real vector가 되는 방법이죠. 여기에 normalization을 위해 l2_norm(s)나 word 개수 (여기서는 4)로 s를 나눠 주면 좀 더 reliable한 표현이 될 것입니다.

---

> And here comes the Keras, which is a widely used high-level wrapper for TensorFlow (and other libraries i.e. Theano and CNTK; though not used in general). Since we only use the *Sequential()* for the codes not incorporating the concatenated layers or attention models, only the *layer* will be imported. For the computation of average/weighted F1 score per epoch, an additional class is defined here. Also, considering the imbalance in the class volumes, we obtain the class weight set that is utilized in the training session.

```python
import tensorflow as tf
from keras.backend.tensorflow_backend import set_session
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.1
set_session(tf.Session(config=config))
from keras.models import Sequential
import keras.layers as layers
from keras import optimizers
adam_half = optimizers.Adam(lr=0.0005)
from keras.callbacks import ModelCheckpoint

from keras.callbacks import Callback
from sklearn import metrics
class Metricsf1macro(Callback):
    def on_train_begin(self, logs={}):
        self.val_f1s = []
        self.val_recalls = []
        self.val_precisions = []
        self.val_f1s_w = []
        self.val_recalls_w = []
        self.val_precisions_w = []
    def on_epoch_end(self, epoch, logs={}):
        val_predict = np.asarray(self.model.predict(self.validation_data[0]))
        val_predict = np.argmax(val_predict,axis=1)
        val_targ = self.validation_data[1]
        _val_f1 = metrics.f1_score(val_targ, val_predict, average="macro")
        _val_f1_w = metrics.f1_score(val_targ, val_predict, average="weighted")
        _val_recall = metrics.recall_score(val_targ, val_predict, average="macro")
        _val_recall_w = metrics.recall_score(val_targ, val_predict, average="weighted")
        _val_precision = metrics.precision_score(val_targ, val_predict, average="macro")
        _val_precision_w = metrics.precision_score(val_targ, val_predict, average="weighted")
        self.val_f1s.append(_val_f1)
        self.val_recalls.append(_val_recall)
        self.val_precisions.append(_val_precision)
        self.val_f1s_w.append(_val_f1_w)
        self.val_recalls_w.append(_val_recall_w)
        self.val_precisions_w.append(_val_precision_w)
        print("— val_f1: %f — val_precision: %f — val_recall: %f"%(_val_f1, _val_precision, _val_recall))
        print("— val_f1_w: %f — val_precision_w: %f — val_recall_w: %f"%(_val_f1_w, _val_precision_w, _val_recall_w))

metricsf1macro = Metricsf1macro()

from sklearn.utils import class_weight
class_weights_fci = class_weight.compute_class_weight('balanced', np.unique(fci_label), fci_label)
```

* 본격적으로 Keras를 써볼 때가 왔습니다. 일단 제가 TensorFlow를 제대로 써본 적은 없지만, conctenated layer나 attention model같은 복잡한 시스템을 포함하지 않는 대부분의 모델들에 대해서는 Sequential() 모듈을 import하는 것으로 대부분 구현 가능합니다 ㅎㅎ TF-IDF의 evaluation에서 활용한 average/weighted F1 score을 epoch마다 계산하기 위해 별도의 함수를 정의했으며, class imbalance를 고려하여 weight set을 구했습니다. 다음의 코드에서 활용되는 것을 볼 수 있습니다.

---

> And the Below is the model construction and the evaluation phase. Note that the folder *tutorial* was created in the same directory to save the checkpoint models, recording F1 scores and the accuracy. It is quite surprising that a simple summation boosted the accuracy and the F1 score by a large factor, even considering that the concept of making up the sentence vector is fundamentally identical to those of one-hot encoding and TF-IDF!

```properties
def validate_nn(result,y,hidden_dim,cw,filename):
    model = Sequential()
    model.add(layers.Dense(hidden_dim, activation = 'relu', input_dim=len(result[0])))
    model.add(layers.Dense(int(max(y))+1, activation='softmax'))
    model.summary()
    model.compile(optimizer=adam_half, loss="sparse_categorical_crossentropy", metrics=['accuracy'])
    filepath=filename+"-{epoch:02d}-{val_acc:.4f}.hdf5"
    checkpoint = ModelCheckpoint(filepath, monitor='val_acc', verbose=1, mode='max')
    callbacks_list = [metricsf1macro,checkpoint]
    model.fit(result,y,validation_split=0.1,epochs=30,batch_size=16,callbacks=callbacks_list,class_weight=cw)

validate_nn(fci_nn,fci_label,128,class_weights_fci,'model/tutorial/nn')

>>> validate_nn(fci_nn,fci_label,128,class_weights_fci,'model/tutorial/nn')
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
dense_8 (Dense)              (None, 128)               12928     
_________________________________________________________________
dense_9 (Dense)              (None, 7)                 903       
=================================================================
Total params: 13,831
Trainable params: 13,831
Non-trainable params: 0
_________________________________________________________________
Train on 51684 samples, validate on 5743 samples
Epoch 1/30
51568/51684 [============================>.] - ETA: 0s - loss: 0.8395 - acc: 0.7203— val_f1: 0.529105 — val_precision: 0.574037 — val_recall: 0.514271
— val_f1_w: 0.728250 — val_precision_w: 0.717994 — val_recall_w: 0.746648
51684/51684 [==============================] - 6s 125us/step - loss: 0.8396 - acc: 0.7202 - val_loss: 0.7489 - val_acc: 0.7466
Epoch 2/30
51344/51684 [============================>.] - ETA: 0s - loss: 0.7221 - acc: 0.7553— val_f1: 0.551772 — val_precision: 0.649961 — val_recall: 0.539155
— val_f1_w: 0.738922 — val_precision_w: 0.742050 — val_recall_w: 0.755877
51684/51684 [==============================] - 6s 119us/step - loss: 0.7218 - acc: 0.7552 - val_loss: 0.7044 - val_acc: 0.7559
Epoch 3/30
51344/51684 [============================>.] - ETA: 0s - loss: 0.6854 - acc: 0.7677— val_f1: 0.579180 — val_precision: 0.689582 — val_recall: 0.555189
— val_f1_w: 0.759864 — val_precision_w: 0.768206 — val_recall_w: 0.774856
51684/51684 [==============================] - 6s 121us/step - loss: 0.6856 - acc: 0.7676 - val_loss: 0.6680 - val_acc: 0.7749
Epoch 4/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.6590 - acc: 0.7778— val_f1: 0.578035 — val_precision: 0.707392 — val_recall: 0.548753
— val_f1_w: 0.759995 — val_precision_w: 0.768224 — val_recall_w: 0.776423
51684/51684 [==============================] - 6s 121us/step - loss: 0.6590 - acc: 0.7778 - val_loss: 0.6499 - val_acc: 0.7764
Epoch 5/30
51664/51684 [============================>.] - ETA: 0s - loss: 0.6376 - acc: 0.7843— val_f1: 0.604010 — val_precision: 0.704348 — val_recall: 0.579958
— val_f1_w: 0.773616 — val_precision_w: 0.779264 — val_recall_w: 0.787393
51684/51684 [==============================] - 6s 120us/step - loss: 0.6375 - acc: 0.7843 - val_loss: 0.6299 - val_acc: 0.7874
Epoch 6/30
51600/51684 [============================>.] - ETA: 0s - loss: 0.6201 - acc: 0.7918— val_f1: 0.601519 — val_precision: 0.725732 — val_recall: 0.572787
— val_f1_w: 0.779200 — val_precision_w: 0.788667 — val_recall_w: 0.794184
51684/51684 [==============================] - 6s 121us/step - loss: 0.6199 - acc: 0.7919 - val_loss: 0.6151 - val_acc: 0.7942
Epoch 7/30
51296/51684 [============================>.] - ETA: 0s - loss: 0.6050 - acc: 0.7957— val_f1: 0.614793 — val_precision: 0.710008 — val_recall: 0.585230
— val_f1_w: 0.783822 — val_precision_w: 0.786432 — val_recall_w: 0.796622
51684/51684 [==============================] - 6s 122us/step - loss: 0.6051 - acc: 0.7957 - val_loss: 0.6045 - val_acc: 0.7966
Epoch 8/30
51360/51684 [============================>.] - ETA: 0s - loss: 0.5917 - acc: 0.8013— val_f1: 0.620563 — val_precision: 0.726211 — val_recall: 0.588463
— val_f1_w: 0.790249 — val_precision_w: 0.797681 — val_recall_w: 0.803239
51684/51684 [==============================] - 6s 121us/step - loss: 0.5916 - acc: 0.8013 - val_loss: 0.5926 - val_acc: 0.8032
Epoch 9/30
51568/51684 [============================>.] - ETA: 0s - loss: 0.5803 - acc: 0.8052— val_f1: 0.614362 — val_precision: 0.755863 — val_recall: 0.581659
— val_f1_w: 0.787776 — val_precision_w: 0.808819 — val_recall_w: 0.800104
51684/51684 [==============================] - 6s 120us/step - loss: 0.5801 - acc: 0.8052 - val_loss: 0.5963 - val_acc: 0.8001
Epoch 10/30
51440/51684 [============================>.] - ETA: 0s - loss: 0.5694 - acc: 0.8096— val_f1: 0.630415 — val_precision: 0.731874 — val_recall: 0.597364
— val_f1_w: 0.797703 — val_precision_w: 0.804632 — val_recall_w: 0.808985
51684/51684 [==============================] - 6s 117us/step - loss: 0.5696 - acc: 0.8095 - val_loss: 0.5745 - val_acc: 0.8090
Epoch 11/30
51552/51684 [============================>.] - ETA: 0s - loss: 0.5602 - acc: 0.8131— val_f1: 0.647759 — val_precision: 0.721144 — val_recall: 0.614857
— val_f1_w: 0.801638 — val_precision_w: 0.806856 — val_recall_w: 0.810030
51684/51684 [==============================] - 6s 122us/step - loss: 0.5599 - acc: 0.8132 - val_loss: 0.5724 - val_acc: 0.8100
Epoch 12/30
51488/51684 [============================>.] - ETA: 0s - loss: 0.5504 - acc: 0.8164— val_f1: 0.649932 — val_precision: 0.721899 — val_recall: 0.617662
— val_f1_w: 0.804792 — val_precision_w: 0.806620 — val_recall_w: 0.813164
51684/51684 [==============================] - 6s 122us/step - loss: 0.5508 - acc: 0.8162 - val_loss: 0.5684 - val_acc: 0.8132
Epoch 13/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.5428 - acc: 0.8187— val_f1: 0.630203 — val_precision: 0.752922 — val_recall: 0.597303
— val_f1_w: 0.799447 — val_precision_w: 0.808868 — val_recall_w: 0.811771
51684/51684 [==============================] - 6s 122us/step - loss: 0.5429 - acc: 0.8187 - val_loss: 0.5601 - val_acc: 0.8118
Epoch 14/30
51568/51684 [============================>.] - ETA: 0s - loss: 0.5362 - acc: 0.8222— val_f1: 0.644598 — val_precision: 0.750094 — val_recall: 0.609115
— val_f1_w: 0.805831 — val_precision_w: 0.814307 — val_recall_w: 0.816820
51684/51684 [==============================] - 6s 122us/step - loss: 0.5360 - acc: 0.8223 - val_loss: 0.5551 - val_acc: 0.8168
Epoch 15/30
51392/51684 [============================>.] - ETA: 0s - loss: 0.5291 - acc: 0.8226— val_f1: 0.648176 — val_precision: 0.743461 — val_recall: 0.611869
— val_f1_w: 0.807041 — val_precision_w: 0.814813 — val_recall_w: 0.816298
51684/51684 [==============================] - 6s 122us/step - loss: 0.5288 - acc: 0.8227 - val_loss: 0.5520 - val_acc: 0.8163
Epoch 16/30
51488/51684 [============================>.] - ETA: 0s - loss: 0.5224 - acc: 0.8261— val_f1: 0.649435 — val_precision: 0.740358 — val_recall: 0.612228
— val_f1_w: 0.806029 — val_precision_w: 0.813032 — val_recall_w: 0.815776
51684/51684 [==============================] - 6s 123us/step - loss: 0.5221 - acc: 0.8262 - val_loss: 0.5482 - val_acc: 0.8158
Epoch 17/30
51472/51684 [============================>.] - ETA: 0s - loss: 0.5163 - acc: 0.8290— val_f1: 0.653423 — val_precision: 0.753656 — val_recall: 0.614267
— val_f1_w: 0.811074 — val_precision_w: 0.817580 — val_recall_w: 0.820651
51684/51684 [==============================] - 6s 120us/step - loss: 0.5163 - acc: 0.8291 - val_loss: 0.5456 - val_acc: 0.8207
Epoch 18/30
51584/51684 [============================>.] - ETA: 0s - loss: 0.5106 - acc: 0.8299— val_f1: 0.654337 — val_precision: 0.747464 — val_recall: 0.621937
— val_f1_w: 0.811604 — val_precision_w: 0.817200 — val_recall_w: 0.821348
51684/51684 [==============================] - 6s 120us/step - loss: 0.5106 - acc: 0.8299 - val_loss: 0.5420 - val_acc: 0.8213
Epoch 19/30
51408/51684 [============================>.] - ETA: 0s - loss: 0.5053 - acc: 0.8319— val_f1: 0.647264 — val_precision: 0.752377 — val_recall: 0.609879
— val_f1_w: 0.800680 — val_precision_w: 0.805022 — val_recall_w: 0.811248
51684/51684 [==============================] - 6s 118us/step - loss: 0.5053 - acc: 0.8319 - val_loss: 0.5506 - val_acc: 0.8112
Epoch 20/30
51328/51684 [============================>.] - ETA: 0s - loss: 0.4988 - acc: 0.8343— val_f1: 0.671217 — val_precision: 0.737495 — val_recall: 0.640099
— val_f1_w: 0.814082 — val_precision_w: 0.817465 — val_recall_w: 0.821174
51684/51684 [==============================] - 6s 117us/step - loss: 0.4997 - acc: 0.8341 - val_loss: 0.5360 - val_acc: 0.8212
Epoch 21/30
51264/51684 [============================>.] - ETA: 0s - loss: 0.4944 - acc: 0.8366— val_f1: 0.664458 — val_precision: 0.740520 — val_recall: 0.632045
— val_f1_w: 0.812932 — val_precision_w: 0.817124 — val_recall_w: 0.821696
51684/51684 [==============================] - 6s 120us/step - loss: 0.4946 - acc: 0.8366 - val_loss: 0.5365 - val_acc: 0.8217
Epoch 22/30
51392/51684 [============================>.] - ETA: 0s - loss: 0.4904 - acc: 0.8366— val_f1: 0.652959 — val_precision: 0.728956 — val_recall: 0.616198
— val_f1_w: 0.808515 — val_precision_w: 0.812157 — val_recall_w: 0.817517
51684/51684 [==============================] - 6s 121us/step - loss: 0.4905 - acc: 0.8366 - val_loss: 0.5426 - val_acc: 0.8175
Epoch 23/30
51312/51684 [============================>.] - ETA: 0s - loss: 0.4854 - acc: 0.8395— val_f1: 0.683792 — val_precision: 0.733536 — val_recall: 0.653159
— val_f1_w: 0.818489 — val_precision_w: 0.819629 — val_recall_w: 0.823263
51684/51684 [==============================] - 6s 122us/step - loss: 0.4857 - acc: 0.8395 - val_loss: 0.5363 - val_acc: 0.8233
Epoch 24/30
51264/51684 [============================>.] - ETA: 0s - loss: 0.4815 - acc: 0.8404— val_f1: 0.670460 — val_precision: 0.751412 — val_recall: 0.632046
— val_f1_w: 0.814917 — val_precision_w: 0.816401 — val_recall_w: 0.823263
51684/51684 [==============================] - 6s 119us/step - loss: 0.4813 - acc: 0.8403 - val_loss: 0.5349 - val_acc: 0.8233
Epoch 25/30
51456/51684 [============================>.] - ETA: 0s - loss: 0.4778 - acc: 0.8422— val_f1: 0.660602 — val_precision: 0.767336 — val_recall: 0.616146
— val_f1_w: 0.817814 — val_precision_w: 0.826908 — val_recall_w: 0.827442
51684/51684 [==============================] - 6s 121us/step - loss: 0.4774 - acc: 0.8423 - val_loss: 0.5352 - val_acc: 0.8274
Epoch 26/30
51360/51684 [============================>.] - ETA: 0s - loss: 0.4739 - acc: 0.8426— val_f1: 0.665963 — val_precision: 0.774288 — val_recall: 0.624489
— val_f1_w: 0.818409 — val_precision_w: 0.826185 — val_recall_w: 0.827442
51684/51684 [==============================] - 6s 118us/step - loss: 0.4736 - acc: 0.8427 - val_loss: 0.5307 - val_acc: 0.8274
Epoch 27/30
51296/51684 [============================>.] - ETA: 0s - loss: 0.4697 - acc: 0.8438— val_f1: 0.683299 — val_precision: 0.737117 — val_recall: 0.652833
— val_f1_w: 0.822906 — val_precision_w: 0.823076 — val_recall_w: 0.828661
51684/51684 [==============================] - 6s 120us/step - loss: 0.4698 - acc: 0.8437 - val_loss: 0.5255 - val_acc: 0.8287
```

* 모델 construction과 training-evaluation입니다. 매 checkpoint에서 모델들이 tutorial이라는 폴더에 저장되어야 하니, 미리 만들어 두어야겠죠 ㅎㅎ TF-IDF의 결과들이 그렇게 만족스럽지는 못했다는 걸 생각하면, 괄목할 만한 성장입니다. accuracy도 올랐고, F1의 평균값 (val_f1)도 상당한 수준으로 상승했네요. one-hot vector을 만들 때 그랬던 것처럼 그냥 구성요소들을 더했을 뿐인데 ...?

* 겨우 100차원인 벡터들을 더해서 뭘 표현할 수 있을까? 싶은 분들도 분명 계실 겁니다. 하지만, 두 가지를 상기할 필요가 있습니다. (1) word vector들은 one-hot vector들처럼 equivalent하지 않고, 특정 기준에 의해 training되었다 - 즉 그 자체로 어떤 의미를 지니고 있다. (2) 벡터들의 합으로 얻는 벡터 역시 100dim 공간에 표현될 수 있으며, 100dim은 그 방향만 해도 2^100 개 이상을 나타낼 수 있을 정도로 꽤나 많은 것을 표현할 수 있다.

---

> This kind of sentence encoding gives us quite a rich representation of the sentences in the sense that the 100-dimensional vector itself yields a variety of values. This might be advantageous for tasks such as sentiment analysis, in which the inference largely relies on some polarity items or emotion words. However, we should notice that a simple summation does not say anything about the distributional or sequential information the sentence possesses; for instance, “You haven’t done it at all” and “Haven’t you done it at all” share same word composition but the intention clearly differs. The same kind of problem is more critical in Korean, due to the language being scrambling.

* 이런 방식의 sentence vector 만들기는 sentiment classification 같은 task에서는 꽤나 좋은 성능을 냅니다. sentiment는 주로 단어 내의 어떤 polarity 및 subjectivity가 있는 단어에 의해 형성될 가능성이 높은데, 더하는 것만으로도 어떤 word가 있다는 것을 classifier가 알게 하기엔 충분하기 때문이죠. 하지만, summation의 단점은 분포나 순서를 고려할 수 없다는 겁니다. 예컨대, “You haven’t done it at all”과 “Haven’t you done it at all”은 그 단어 구성은 같아도 (capital 여부는 무시합시다) 전달하는 의미는 전혀 다르죠. word vector의 summation이 아니라 concatenation으로 한다면 이런 일을 예방할 수 있겠습니다만, 뭔가 임시방편적인 처방이고 결국 다시 저차원 임베딩이 아니게 되어버리겠죠. 이런 문제를 어떻게 하면 해결할 수 있을까요?

## 6. CNN-based sentence classification

> [**Convolutional neural network**](https://ieeexplore.ieee.org/abstract/document/726791) (CNN), originally developed by LeCun, is a widely used neural network system which was primarily suggested for image processing (handwriting recognition). It roughly resembles the perception process of a visual system, summarizing a given image into an abstract values via repititive application of convolution and pooling. 

<p align="center">
    <image src="https://github.com/warnikchow/dlk2nlp/blob/master/image/alexnet2.png" width="700"><br/>
    (image from https://jeremykarnowski.wordpress.com/2015/07/15/alexnet-visualization/)

* AI를 공부하시는 분이라면 convolutional neural network, 즉 CNN을 한번쯤 들어보셨을 겁니다. 초기에 르쿤 등등에 의해 연구되고, 6-7년 전쯤부터 폭발적으로 성장하여 지금은 이미지 관련 태스크라면 베이스라인 혹은 그 베리에이션으로 인용된다는 것두요. 그러한 이력 덕분인지, NLP에 CNN을 사용한다고 하면 의아해하는 경우가 있더군요. 저도 사실 익숙해져서 그렇지, 다시 첨부터 써보라고 하면 이게 무슨 소리야 싶을지도 모르겠네요ㅎㅎ

* 제가 이미지에 CNN을 사용해본 적은 거의 없지만, 쉽게 말해 raw data를 부분부분 보고 그 정보를 추상화하는 과정을 여러번 거친다, 라고 생각하고 있습니다. 아주 러프하게요. 그것이 이미지에 처음 적용된 것이죠. 하지만 사실 정보의 추상화란 이미지에만 적용될 이유는 없습니다. 그래서 저는 cnn을 distributional information의 summarizer로 표현합니다. 어디에 무엇이 있는지 아주 간단하게 요약해 주는.

---

> However, since the very classic breakthrough of [Kim 2014](https://arxiv.org/abs/1408.5882), CNN has been widely used in the text processing, understanding the word vector sequence as a single channel image. Different from the previous approaches which incorporate all the words in the sentence into a single vector, the featurization for CNN has its limitation in the volume. Thus, hereby we restrict the maximum length of the morpheme sequence to 30, with zero-padding for the short utterances. Taking into account the head-finality of Korean, we've decided to place the word vectors on the right side of the matrix. That is, for the long utterances, only the last 30 morphemes are utilized.

```python
def featurize_cnn(corpus,wdim,maxlen):
    conv_total = np.zeros((len(corpus),maxlen,wdim,1))
    for i in range(len(corpus)):
        if i%1000 ==0:
            print(i)
        s = corpus[i]
        for j in range(len(s)):
            if s[-j-1] in model_ft and j < maxlen:
                conv_total[i][-j-1,:,0] = model_ft[s[-j-1]]
    return conv_total
    
fci_conv = featurize_cnn(fci_sp_token_train+fci_sp_token_test,100,30)
```

<p align="center">
    <image src="https://github.com/warnikchow/dlk2nlp/blob/master/image/ykim14.png" width="700"><br/>
    (image from [Kim 2014](https://arxiv.org/abs/1408.5882))

* 이것이 문장 분류에 어떻게 사용되느냐? 가장 먼저 거치는 과정은 쉽게 말해 문장을 그림처럼 바꾸는 겁니다. 즉, 단일 채널 matrix를 만드는 거죠 (그림은 보통 rgb의 3 channel). 우린 sentence matrix란 걸 논한 적 없으니 word vector들로 어떻게 해 봐야 될 텐데, word vector나 TF-IDF를 가지고는 듬성듬성하게 nonzero가 박혀 있는 것들밖에 만들지 못할 테죠. 애초에 값에 대한 위치 bias가 없는 녀석들이니 순서(order)적인 것 외에 아무 정보도 CNN에 주지는 못할 겁니다.

* 이 때 다시 등장하는 것이 앞서 언급한 word2vec입니다. 문장을 수치화해 넣을 수 있는 일종의 고정된 사이즈의 도화지가 있다고 생각해 봅시다. 예컨대 100 x 30정도의? 거기에 100-dim word vector 30개를 padding해 넣는 겁니다. 물론 문장 길이가 30이 되지 않을 수도 있지요. 그러면 빈 부분은 0으로 채웁니다. 진짜 없으니까요. 문장이 더 길다면? 자릅니다. 물론 이 부분은 '문장 최대길이'를 조사해서 적절히 설정하면 될 일입니다 (물론 이렇게 하지 않고 모두 보존하는 방법도 있겠습니다만, 일단 여기선 다루지 않겠습니다). 이 과정에서 한국어의 head-finality를 고려하여 문장은 오른쪽에 치우치게 배열하도록 결정하였습니다. 한국어는 역시 끝까지 들어봐야 하니까, 자르더라도 끝은 남겨야죠!

---

> There are so many types of convolutional networks out there (LeNet, AlexNet, VGG, YOLO ...), however the sentence classification does not require such deep and wide networks. The specification used for the implementation is quite simple; two convolutional layers with the window width 3 and a max pooling layer between with the size (2,1). For the first conv-layer, the window size is (3,100) and for the second (3,1), since the information was abstracted and max-pooled to make up a single vector.

```python
def validate_cnn(result,y,filters,hidden_dim,cw,filename):
    model = Sequential()
    model.add(layers.Conv2D(filters,(3,len(result[0][0])),activation= 'relu',input_shape = (len(result[0]),len(result[0][0]),1)))
    model.add(layers.MaxPooling2D((2,1)))
    model.add(layers.Conv2D(filters,(3,1),activation='relu'))
    model.add(layers.Flatten())
    model.add(layers.Dense(hidden_dim,activation='relu'))
    model.add(layers.Dense(int(max(y)+1), activation='softmax'))
    model.summary()
    model.compile(optimizer=adam_half, loss="sparse_categorical_crossentropy", metrics=["accuracy"])
    filepath=filename+"-{epoch:02d}-{val_acc:.4f}.hdf5"
    checkpoint = ModelCheckpoint(filepath, monitor='val_acc', verbose=1, mode='max')
    callbacks_list = [metricsf1macro,checkpoint]
    model.fit(result,y,validation_split=0.1,epochs=30,batch_size=16,callbacks=callbacks_list,class_weight=cw)

validate_cnn(fci_conv,fci_label,32,128,class_weights_fci,'model/tutorial/conv')
```

* 일단 image를 cnn에 적용하는 과정을 패러미터화하면, 채널, 필터, 컨벌루션레이어, 윈도우, 풀링 정도로 요약할 수 있습니다. 채널은 앞서 말했듯 rgb 같이 몇 개의 요소로 나타내냐이며 필터는 얼마나 병렬로 처리할거냐, 컨벌루션레이어 수는 추상화 과정을 몇번 거칠거냐, 윈도우는 어떤 식으로 각 컨벌루션 레이어를 훑을거냐, 풀링은 컨벌루션 레이어를 훑은 값들에서 중요한 요소들을 어떻게 취사선택할거냐? 이정도로 나타낼 수 있겠네요.

* 이미지의 cnn은 그래서 일반적으로 3채널, 많은 필터 (>64?), 다층 컨벌루션 레이어, 상하좌우로 stride되는 3 by 3 혹은 5 by 5 window 등으로 요약될 수 있습니다. 물론 alexnet, vgg, yolo 등 다양한 아키텍쳐들이 있고, 모두 특색이 있겠지만, 기본적으론 저렇습니다. 하지만 word vector sequence에서 상하좌우로 움직이는 window가 어떤 의미가 있을까요? 우리는 100dim의 벡터 각 엔트리에 어떤 성질의 성분들이 자리잡고 있는지 알지 못하며, 굳이 그런 성질을 지정해줄 필요도 느끼지 못하였습니다. 이미지는 두차원 모두가 semantic을 포함하지만, sentence에서 semantic이 의미가 있는 방향은 word vector가 pad되는 방향이니까요.

* 그래서 저는 sentence의 cnn에선 3 by 100 혹은 5 by 100 window를 사용합니다. 결론적으론 2D convolution이 1D처럼 돼버리긴 합니다만, 역설적으로 문장을 그림이 아니라 문장처럼 볼 수 있는 방법이 되는 것 같아요. Word2vec의 결과물로 나온 그 단어의 vector의 특색을 결정짓는 entry를 가장 왜곡 없이 전달해줄 수 있는 방법이라고 저는 보고 있습니다. 그 이후의 max-pooling과 추가적인 convolution 아키텍쳐는 개인의 선택에 달렸지만요. Boolean distributional semantics처럼 word vector의 각 entry가 어떤 의미를 갖는다면 모르겠지만, 그렇지 않다면 문장은 문장처럼 읽는 것이 cnn에 있어서도 효과적이지 않나? 라는 것이 저의 생각입니다. (물론 반례 및 피드백은 언제나 환영입니다!)

---

> The result is encouraging! Although there was a little improvement in F1 score, we had about a 20% RRER for the accuracy (regarding the model with the best performance). The CNN-based featurization and classification of the sentence shows quite satisfactory result with a very fast training. However, the architecture does not seem to still convey the correlation between the non-consecutive components. The recurrent neural network covers such characteristics, with a sequence-based approach.

```properties
# CONSOLE RESULT
>>> validate_cnn(fci_conv,fci_label,32,128,class_weights_fci,'model/tutorial/conv')
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d_1 (Conv2D)            (None, 28, 1, 32)         9632      
_________________________________________________________________
max_pooling2d_1 (MaxPooling2 (None, 14, 1, 32)         0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 12, 1, 32)         3104      
_________________________________________________________________
flatten_1 (Flatten)          (None, 384)               0         
_________________________________________________________________
dense_1 (Dense)              (None, 128)               49280     
_________________________________________________________________
dense_2 (Dense)              (None, 7)                 903       
=================================================================
Total params: 62,919
Trainable params: 62,919
Non-trainable params: 0
_________________________________________________________________
Train on 51684 samples, validate on 5743 samples
Epoch 1/30
51600/51684 [============================>.] - ETA: 0s - loss: 0.6353 - acc: 0.7863— val_f1: 0.577525 — val_precision: 0.721343 — val_recall: 0.562436
— val_f1_w: 0.811520 — val_precision_w: 0.814879 — val_recall_w: 0.832666
51684/51684 [==============================] - 19s 377us/step - loss: 0.6350 - acc: 0.7864 - val_loss: 0.4987 - val_acc: 0.8327
Epoch 2/30
51616/51684 [============================>.] - ETA: 0s - loss: 0.4569 - acc: 0.8482— val_f1: 0.661298 — val_precision: 0.704766 — val_recall: 0.644935
— val_f1_w: 0.829482 — val_precision_w: 0.832600 — val_recall_w: 0.836148
51684/51684 [==============================] - 19s 375us/step - loss: 0.4569 - acc: 0.8482 - val_loss: 0.4827 - val_acc: 0.8361
Epoch 3/30
51568/51684 [============================>.] - ETA: 0s - loss: 0.4069 - acc: 0.8628— val_f1: 0.685981 — val_precision: 0.711728 — val_recall: 0.674920
— val_f1_w: 0.837491 — val_precision_w: 0.840673 — val_recall_w: 0.840327
51684/51684 [==============================] - 19s 373us/step - loss: 0.4067 - acc: 0.8628 - val_loss: 0.4642 - val_acc: 0.8403
Epoch 4/30
51568/51684 [============================>.] - ETA: 0s - loss: 0.3732 - acc: 0.8734— val_f1: 0.694238 — val_precision: 0.743687 — val_recall: 0.666121
— val_f1_w: 0.849517 — val_precision_w: 0.848548 — val_recall_w: 0.854954
51684/51684 [==============================] - 19s 377us/step - loss: 0.3732 - acc: 0.8734 - val_loss: 0.4362 - val_acc: 0.8550
Epoch 5/30
51616/51684 [============================>.] - ETA: 0s - loss: 0.3467 - acc: 0.8810— val_f1: 0.700670 — val_precision: 0.759441 — val_recall: 0.668912
— val_f1_w: 0.850824 — val_precision_w: 0.851885 — val_recall_w: 0.858262
51684/51684 [==============================] - 19s 375us/step - loss: 0.3469 - acc: 0.8809 - val_loss: 0.4275 - val_acc: 0.8583
Epoch 6/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.3264 - acc: 0.8880— val_f1: 0.698527 — val_precision: 0.748593 — val_recall: 0.678563
— val_f1_w: 0.853043 — val_precision_w: 0.852584 — val_recall_w: 0.860700
51684/51684 [==============================] - 19s 371us/step - loss: 0.3264 - acc: 0.8880 - val_loss: 0.4331 - val_acc: 0.8607
```

* 결과입니다. F1 score에서는 그렇게 큰 향상을 얻지는 못했으나, accuracy는 0.8287에서 0.8607로, error rate가 약 17%에서 14%로 감소하며 약 20%의 에러율 감소 (RRER)을 보였습니다. 아무래도 distributional한 information을 반영하는 것이, 단순히 existence를 체크하는 것보다는 더 정확하겠죠.

* 이상으로 distributional information의 가장 효과적인 summarizer 중 하나에 대하여 살펴봤습니다. 하지만 그 단점은, non-consecutive한 성분들 간의 상관관계를 설명하기 쉽지 않다는 점이죠. 다음 편부터 설명되는 rnn은 그러한 점들을 보완합니다.

## 7. RNN (BiLSTM)-based sentence classification

> **Recurrent neural network (RNN)**, which was suggested originally in the late 20th century, is a representative network that reflects the sequential information in the numerical summarization. Due to the high-computation issue, its materialization has recently been possible with the help of modern computing systems (e.g. GPU boost-up). The problem of vanishing gradient has been partially solved by [**long short-term memory (LSTM)**](https://www.mitpressjournals.org/doi/abs/10.1162/neco.1997.9.8.1735), whose direction-bias was improved along with the bidirectional sequencing (BiLSTM).

<p align="center">
    <image src="https://github.com/warnikchow/dlk2nlp/blob/master/image/rnn.jpg" width="700"><br/>
    (image from https://aikorea.org/blog/rnn-tutorial-1/)

* RNN, 즉 recurrent neural network란 time-series의 input을 받아, 그에 대한 latent information을 담고 있는 hidden layer sequence를 생성하되, 특정 시점에서의 hidden layer가 그 시점에서의 input data와 이전 시점의 hidden layer로부터 연산되는 알고리듬입니다. hidden Markov model (HMM)과 그 컨셉은 유사하지만 바로 이전 시점에만 영향받지 않고, 앞서 있는 데이터 모두의 정보를 뒷부분의 hidden layer에 반영한다는 장점이 있죠. 쉽게 말해 sequential data의 summarization이라고 보면 될 것 같네요.

* rnn을 트레이닝하는 과정 역시 일반적인 mlp나 cnn에서와 마찬가지로 back-propagation을 이용하게 되는데요, 이 과정에서 vanishing gradient의 문제가 발생하게 됩니다. 너무 많은 정보들이 encoding되다 보니 패러미터가 폭발해 버리는 겁니다. 사람은 이와 다르게, 문장이나 글이 길어지게 되면 너무 멀리 떨어져 있는 정보는 잊어버리죠 :D 뭐 그게 꼭 좋은 건 아니겠지만서두, 정보 과잉을 방지해주거나 뭐 그렇지 않겠습니까? 그런 컨셉으로 나온 것이 적당히 forget gate를 추가한, 1997년의 long short-term memory (LSTM) 입니다. lstm이 앞부분의 정보를 반영하지 못한다는 단점을 보완하기 위해 제시된 것이, lstm을 양방향으로 (처음에서 시작해서 끝으로, 끝에서 시작해서 처음으로) 하여 얻은 hidden layer sequences를 augment한 Bidirectional lstm도 같은 해에 제시되었구요. 생각보다 옛날인데 왜 이제 와서 유행하게 됐냐구요? 계산량이 엄청나기 때문이죠... 갓비디아 짱짱컴퍼니

---

> In this tutorial, we utilize the BiLSTM structure. The featurization is almost identical to the case of CNN, except that the channel information is omitted. To say short, for the same data (ignoring the channel number), CNN extracts locally notable features and RNN extracts the relations reflected in the sequential arrangement.

```python
def featurize_rnn(corpus,wdim,maxlen):
    rnn_total = np.zeros((len(corpus),maxlen,wdim))
    for i in range(len(corpus)):
        if i%1000 ==0:
            print(i)
        s = corpus[i]
        for j in range(len(s)):
            if s[-j-1] in model_ft and j < maxlen:
                rnn_total[i][-j-1,:] = model_ft[s[-j-1]]
    return rnn_total

fci_rec = featurize_rnn(fci_sp_token_train+fci_sp_token_test,100,30)

from keras.layers import LSTM
from keras.layers import Bidirectional

def validate_bilstm(result,y,hidden_lstm,hidden_dim,cw,filename):
    model = Sequential()
    model.add(Bidirectional(LSTM(hidden_lstm), input_shape=(len(result[0]), len(result[0][0]))))
    model.add(layers.Dense(hidden_dim, activation='relu'))
    model.add(layers.Dense(int(max(y)+1), activation='softmax'))
    model.summary()
    model.compile(optimizer=adam_half, loss="sparse_categorical_crossentropy", metrics=["accuracy"])
    filepath=filename+"-{epoch:02d}-{val_acc:.4f}.hdf5"
    checkpoint = ModelCheckpoint(filepath, monitor='val_acc', verbose=1, mode='max')
    callbacks_list = [metricsf1macro,checkpoint]
    model.fit(result,y,validation_split=0.1,epochs=30,batch_size=16,callbacks=callbacks_list,class_weight=cw)

validate_bilstm(fci_rec,fci_label,32,128,class_weights_fci,'model/tutorial/rec')
```

* 본 튜토리얼에서는 가장 무난히 적용할 수 있는 BiLSTM을 사용합니다. LSTM의 문제는 문장이 길어질수록 앞부분의 내용을 반영하지 못할 수 있다는 것인데, BiLSTM은 LSTM을 양방향으로 돌린 두 sequence를 hidden layer sequence 하나로 concatenate하면서 어느 정도 그런 점을 보완할 수 있게 합니다. CNN과 간단히 차이를 말해 보자면, CNN은 global하게 본 후 local하게 중요한 피쳐들을 뽑아내어 추상화시키는 것이고, RNN (BiLSTM)은 sequential한 배열에 내재한 관계를 추상화한다고 보면 될 것 같습니다. 

---

> We obtained another boost in performance, especially very high in F1 score. This implies that our task which deals with the intention of Korean sentences is largely influenced by the word order. This may have been an important factor in identifying the rhetoricalness, for instance, '뭐 해 지금 (what / do / now, *what (the hell) are you doing now*)' sounds much more rhetorical than '지금 뭐 해 (now / what / do, *what are you doing now*)', in view of native. However, so far we've conducted the experiments given the morpheme decomposition. Will the real semantics NOT be embedded in the characters? Well, before that, how should **character** be defined in Korean, which has distinguished morpho-syllabic blocks (which are different from alphabets) as a unit of character? 

```properties
# CONSOLE RESULT
>>> validate_bilstm(fci_rec,fci_label,32,128,class_weights_fci,'model/tutorial/rec')
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
bidirectional_2 (Bidirection (None, 64)                34048     
_________________________________________________________________
dense_4 (Dense)              (None, 128)               8320      
_________________________________________________________________
dense_5 (Dense)              (None, 7)                 903       
=================================================================
Total params: 43,271
Trainable params: 43,271
Non-trainable params: 0
_________________________________________________________________

Train on 51684 samples, validate on 5743 samples
Epoch 1/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.5610 - acc: 0.8128— val_f1: 0.671337 — val_precision: 0.721156 — val_recall: 0.648963
— val_f1_w: 0.836127 — val_precision_w: 0.835080 — val_recall_w: 0.843287
51684/51684 [==============================] - 81s 2ms/step - loss: 0.5610 - acc: 0.8128 - val_loss: 0.4706 - val_acc: 0.8433
Epoch 2/30
51664/51684 [============================>.] - ETA: 0s - loss: 0.4385 - acc: 0.8521— val_f1: 0.692261 — val_precision: 0.739367 — val_recall: 0.666227
— val_f1_w: 0.846166 — val_precision_w: 0.844998 — val_recall_w: 0.851820
51684/51684 [==============================] - 79s 2ms/step - loss: 0.4385 - acc: 0.8521 - val_loss: 0.4329 - val_acc: 0.8518
Epoch 3/30
51664/51684 [============================>.] - ETA: 0s - loss: 0.3994 - acc: 0.8634— val_f1: 0.697970 — val_precision: 0.777117 — val_recall: 0.659569
— val_f1_w: 0.852976 — val_precision_w: 0.854659 — val_recall_w: 0.861048
51684/51684 [==============================] - 80s 2ms/step - loss: 0.3994 - acc: 0.8635 - val_loss: 0.4148 - val_acc: 0.8610
Epoch 4/30
51648/51684 [============================>.] - ETA: 0s - loss: 0.3715 - acc: 0.8731— val_f1: 0.726711 — val_precision: 0.769367 — val_recall: 0.700206
— val_f1_w: 0.861842 — val_precision_w: 0.861756 — val_recall_w: 0.865575
51684/51684 [==============================] - 79s 2ms/step - loss: 0.3717 - acc: 0.8729 - val_loss: 0.4016 - val_acc: 0.8656
Epoch 5/30
51648/51684 [============================>.] - ETA: 0s - loss: 0.3510 - acc: 0.8804— val_f1: 0.732919 — val_precision: 0.756133 — val_recall: 0.714735
— val_f1_w: 0.865283 — val_precision_w: 0.864796 — val_recall_w: 0.868187
51684/51684 [==============================] - 79s 2ms/step - loss: 0.3511 - acc: 0.8804 - val_loss: 0.3880 - val_acc: 0.8682
Epoch 6/30
51664/51684 [============================>.] - ETA: 0s - loss: 0.3333 - acc: 0.8867— val_f1: 0.744263 — val_precision: 0.742123 — val_recall: 0.752202
— val_f1_w: 0.866932 — val_precision_w: 0.871576 — val_recall_w: 0.863660
51684/51684 [==============================] - 79s 2ms/step - loss: 0.3333 - acc: 0.8867 - val_loss: 0.3944 - val_acc: 0.8637
Epoch 7/30
51664/51684 [============================>.] - ETA: 0s - loss: 0.3170 - acc: 0.8913— val_f1: 0.729010 — val_precision: 0.774354 — val_recall: 0.700533
— val_f1_w: 0.865218 — val_precision_w: 0.863544 — val_recall_w: 0.870625
51684/51684 [==============================] - 80s 2ms/step - loss: 0.3169 - acc: 0.8914 - val_loss: 0.3909 - val_acc: 0.8706
Epoch 8/30
51648/51684 [============================>.] - ETA: 0s - loss: 0.3040 - acc: 0.8954— val_f1: 0.739023 — val_precision: 0.768832 — val_recall: 0.719142
— val_f1_w: 0.867667 — val_precision_w: 0.870100 — val_recall_w: 0.869058
51684/51684 [==============================] - 78s 2ms/step - loss: 0.3039 - acc: 0.8955 - val_loss: 0.3950 - val_acc: 0.8691
Epoch 9/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.2903 - acc: 0.9009— val_f1: 0.751430 — val_precision: 0.769792 — val_recall: 0.739712
— val_f1_w: 0.874110 — val_precision_w: 0.875341 — val_recall_w: 0.874108
51684/51684 [==============================] - 83s 2ms/step - loss: 0.2903 - acc: 0.9010 - val_loss: 0.3772 - val_acc: 0.8741
Epoch 10/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.2779 - acc: 0.9039— val_f1: 0.752393 — val_precision: 0.772166 — val_recall: 0.742639
— val_f1_w: 0.871297 — val_precision_w: 0.871833 — val_recall_w: 0.874282
51684/51684 [==============================] - 83s 2ms/step - loss: 0.2779 - acc: 0.9039 - val_loss: 0.3825 - val_acc: 0.8743
Epoch 11/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.2663 - acc: 0.9100— val_f1: 0.736928 — val_precision: 0.770432 — val_recall: 0.721984
— val_f1_w: 0.862387 — val_precision_w: 0.866533 — val_recall_w: 0.862789
51684/51684 [==============================] - 83s 2ms/step - loss: 0.2663 - acc: 0.9100 - val_loss: 0.4140 - val_acc: 0.8628
Epoch 12/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.2552 - acc: 0.9120— val_f1: 0.748533 — val_precision: 0.784169 — val_recall: 0.726216
— val_f1_w: 0.874515 — val_precision_w: 0.875506 — val_recall_w: 0.876023
51684/51684 [==============================] - 82s 2ms/step - loss: 0.2552 - acc: 0.9120 - val_loss: 0.3885 - val_acc: 0.8760
Epoch 13/30
51680/51684 [============================>.] - ETA: 0s - loss: 0.2445 - acc: 0.9152— val_f1: 0.753067 — val_precision: 0.777666 — val_recall: 0.742513
— val_f1_w: 0.875957 — val_precision_w: 0.877898 — val_recall_w: 0.878983
51684/51684 [==============================] - 82s 2ms/step - loss: 0.2445 - acc: 0.9152 - val_loss: 0.3942 - val_acc: 0.8790
```

* 놀랍게도 정확도가 다시 한번 올랐고, F1 score가 상당한 수준으로 올랐습니다. 평균치가 저 정도 올랐다는 것은, 이제 아무리 F1 score가 안 좋아도 0.5 정도는 된다는 걸 의미한다고 봐도 무방할 것 같습니다. 처음에 TF-IDF로 코드를 돌릴 때 intonation-dependent utterances (마지막 case) 에 대해 상당히 낮은 F1 score가 나왔던 것 같은데, 해당 태스크에서 상대적으로 Recall (재현률)을 높게 하여 false alarm을 울리는 것이 false positive보다는 낫다는 점을 고려하면 나쁘지 않은 결과입니다. 그런데 한 가지 간과한 것이 있습니다. 지금까지 우리는 열심히 morpheme들을 가지고 놀았는데, 과연 이것을 character-level로 끊어 보면 어떻게 될까요? 아니, 그 전에 우선, 한국어에서 character을 어떻게 정의하는 것이 바람직할까요?

## 8. Character embedding
## 9. Concatenation of CNN and RNN layers
## 10. BiLSTM Self-attention
