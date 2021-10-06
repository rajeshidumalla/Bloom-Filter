# Bloom Filters

### Setup

In this Pyhton code I am going to install a [bloom_filter](https://github.com/hiway/python-bloom-filter), a Python library which offers an implementations of Bloom filters.  Run the cell below!


```python
!pip install bloom_filter
```

    Collecting bloom_filter
      Using cached bloom_filter-1.3.3-py3-none-any.whl (8.1 kB)
    Installing collected packages: bloom-filter
    Successfully installed bloom-filter-1.3.3


### Data Loading

From the NLTK (Natural Language ToolKit) library, I am importing a large list of English dictionary words, commonly used by the very first spell-checking programs in Unix-like operating systems.


```python
import nltk
nltk.download('words')

from nltk.corpus import words
word_list = words.words()
print(f'Dictionary length: {len(word_list)}')
print(word_list[:15])
```

    [nltk_data] Downloading package words to
    [nltk_data]     /Users/rajeshidumalla/nltk_data...


    Dictionary length: 236736
    ['A', 'a', 'aa', 'aal', 'aalii', 'aam', 'Aani', 'aardvark', 'aardwolf', 'Aaron', 'Aaronic', 'Aaronical', 'Aaronite', 'Aaronitic', 'Aaru']


    [nltk_data]   Unzipping corpora/words.zip.


Then I am loading another dataset from the NLTK Corpora collection: ```movie_reviews```.

The movie reviews are categorized between *positive* and *negative*, so I will construct a list of words (usually called **bag of words**) for each category.


```python
from nltk.corpus import movie_reviews
nltk.download('movie_reviews')

neg_reviews = []
pos_reviews = []

for fileid in movie_reviews.fileids('neg'):
  neg_reviews.extend(movie_reviews.words(fileid))
for fileid in movie_reviews.fileids('pos'):
  pos_reviews.extend(movie_reviews.words(fileid))
```

    [nltk_data] Downloading package movie_reviews to
    [nltk_data]     /Users/rajeshidumalla/nltk_data...
    [nltk_data]   Unzipping corpora/movie_reviews.zip.


And now I am going to develop a very simplistic spell-checker.  By no means I was first think of using it for a real-world use case, but it is an interesting exercise to highlight the strenghts and weaknesses of Bloom Filters!


```python
from bloom_filter import BloomFilter

word_filter = BloomFilter(max_elements=236736)

for word in word_list:
  word_filter.add(word)

word_set = set(word_list)
```

If you executed the cell above, you now have 3 different variables in your scope:

1.   ```word_list```, a Python list containing the English dictionary (in case insensitive order)
2.   ```word_filter```, a Bloom filter where we have already added all the words in the English dictionary
3.   ```word_set```, a [Python set](https://docs.python.org/3.6/library/stdtypes.html#set-types-set-frozenset) built from the same list of words in the English dictionary

Let's inspect the size of each datastructure using the [getsizeof()](https://docs.python.org/3/library/sys.html#sys.getsizeof) method!




```python
from sys import getsizeof
print(f'Size of word_list (in bytes): {getsizeof(word_list)}')

print(f'Size of word_filter (in bytes): {getsizeof(word_filter)}')
print(f'Szie of word_set (in bytes): {getsizeof(word_set)}')
```

    Size of word_list (in bytes): 2115944
    Size of word_filter (in bytes): 48
    Szie of word_set (in bytes): 8388824


You should have noticed how efficient is the Bloom filter in terms of memory footprint!

Now let's find out how fast is the main operation for which we construct Bloom filters: *membership testing*. To do so, we will use the ```%timeit``` IPython magic command, which times the repeated execution of a single Python statement.


```python
%timeit -r 3 "California" in word_list


%timeit "California" in word_filter
#%timeit "California" in word_set
```

    436 µs ± 36.9 µs per loop (mean ± std. dev. of 3 runs, 1000 loops each)
    15.2 µs ± 1.29 µs per loop (mean ± std. dev. of 7 runs, 100000 loops each)


Notice the performance gap between linear search on a list, multiple hash computations in a Bloom filter, and a single hash computation in a native Python ```Set()```.

I now have all the building blocks required to build our spell-checker, and we understand the performance tradeoffs of each datastructure we chose. Write a function that takes as arguments (1) a list of words, and (2) any of the 3 dictionary datastructures we constructed. The function must return the number of words which **do not appear** in the dictionary.


```python
def func(a, ds):
  cnt = 0
  for x in a:
    if x not in ds:
      cnt += 1
  
  print(cnt / len(a))

%timeit func(neg_reviews, word_filter)


```

    0.25797065181509365
    0.25797065181509365
    0.25797065181509365
    0.25797065181509365
    0.25797065181509365
    0.25797065181509365
    0.25797065181509365
    0.25797065181509365
    6.09 s ± 699 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

