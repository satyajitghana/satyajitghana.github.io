title: TorchText New API Changes
date: 2021-05-31 13:00
modified: 2020-05-31 13:00
category: pytorch
tags: api, pytorch 
slug: torchtext-new-api
author: satyajit-ghana
summary: TorchText has now made many of their api's legacy, this post is tracking down the new APIs introduced to torchtext 

# TorchText New API Changes

## Torchtext 0.9.0 release note

Based on the feedback from users, there are several issues existing in torchtext, including

Several components and functionals are unclear and difficult to adopt. For example, Field class couples tokenizer, vocabulary, split, batching and sampling, padding, and numericalization together. The current Field class works as a "black box", and users are confused about what's going on within the class. Instead, those components should be divided into several basic building blocks. This is more consistent with PyTorch core library, which grants users the freedom to build the models and pipelines with orthogonal components.
Incompatible with DataLoader and Sampler in torch.utils.data. The current datasets in torchtext are not compatible with PyTorch core library. Some custom modules/functions in torchtext (e.g. Iterator, Batch, splits) should be replaced by the corresponding modules in torch.utils.data.
New datasets in torchtext.experimental.datasets
We have re-written several datasets in torchtext.experimental.datasets which were using the new abstractions. The old version of the datasets are still available in torchtext.datasets and the new datasets are opt-in.

- Sentiment analysis dataset (#651)
  - IMDB
- Language modeling datasets (#624), including
  - WikiText2
  - WikiText103
  - PennTreebank

### Case study for IMDB dataset

API for new datasets

To load the new datasets, simply call the dataset API, as follow:

```
from torchtext.experimental.datasets import IMDB
train_dataset, test_dataset = IMDB()
```

To specify a tokenizer:

```
from torchtext.data.utils import get_tokenizer
tokenizer = get_tokenizer("spacy")
train_dataset, test_dataset = IMDB(tokenizer=tokenizer)
```


If you just need the test set (must pass a Vocab object!):

```
vocab = train_dataset.get_vocab()
test_dataset, = IMDB(tokenizer=tokenizer, vocab=vocab, data_select='test')
```

Legacy code

The old IMDB dataset is still available in the folder torchtext.datasets. You can use the legacy datasets, as follow:

```
import torchtext.data as data
TEXT = torchtext.data.Field(lower=True, include_lengths=True, batch_first=True)
LABEL = torchtext.data.Field(sequential=False)
train, test = torchtext.datasets.IMDB.splits(TEXT, LABEL)
```

### Difference

With the old pattern, users have to create a Field object including a specific tokenizer. In the new dataset API, user can pass a custom tokenizer directly to the dataset constructor. A custom tokenizer defines the method to convert a string to a list of tokens

from torchtext.data.utils import get_tokenizer

### Old pattern

```
TEXT = torchtext.data.Field(tokenize=get_tokenizer("basic_english"))
```

#### New pattern

```
train_dataset, test_dataset = IMDB(tokenizer=get_tokenizer("spacy"))
```

In the old dataset, vocab object is associated with Field class, which is not flexible enough to accept a pre-trained vocab object. In the new dataset, the vocab object can be obtained by

```
vocab = train_dataset.get_vocab()
new_vocab = torchtext.vocab.Vocab(counter=vocab.freqs, max_size=1000, min_freq=10)
```

and apply to generate other new datasets.

```
from torchtext.experimental.datasets import WikiText2
train_dataset, test_dataset, valid_dataset = WikiText2(vocab=new_vocab)
```

The datasets with the new pattern return a tensor of token IDs, instead of tokens in the old pattern. If users would like to retrieve the tokens, simply use the following command:

```
train_vocab = train_dataset.get_vocab()
```

### label and text are saved as a tuple

```
tokens = [train_vocab.itos[id] for id in train_dataset[0][1]]
```

Unlike the old pattern using BucketIterator.splits, users are encouraged to use torch.utils.data.DataLoader to generate batches of data. You can specify how to batch and pad the samples with a custom function passed to collate_fn. Here is an example to pad sequences with similar lengths and load data through DataLoader. To generate random samples, turn on the shuffle flag in DataLoader. Otherwise, a sequential sampler will be automatically constructed.

### Generate a list of tuples of text length, index, label, text

```
data_len = [(len(txt), idx, label, txt) for idx, (label, txt) in enumerate(train_dataset)]
data_len.sort() # sort by length and pad sequences with similar lengths
```

### Generate the pad id

```
pad_id = train_dataset.get_vocab()['<pad>']
```

### Generate 8x8 batches

### Pad sequences with similar lengths

```
import torch
from torch.utils.data import DataLoader
def pad_data(data):
    # Find max length of the mini-batch
    max_len = max(list(zip(*data))[0])
    label_list = list(zip(*data))[2]
    txt_list = list(zip(*data))[3]
    padded_tensors = torch.stack([torch.cat((txt, \
            torch.tensor([pad_id] * (max_len - len(txt))).long())) \
            for txt in txt_list])
    return padded_tensors, label_list

dataloader = DataLoader(data_len, batch_size=8, collate_fn=pad_data)
for idx, (txt, label) in enumerate(dataloader):
    print(idx, txt.size(), label)
```

Randomly split a dataset into non-overlapping new datasets of given lengths.

```
from torchtext.experimental.datasets import IMDB
train_dataset, test_dataset = IMDB()
train_subset, valid_subset = torch.utils.data.random_split(train_dataset, [15000, 10000])
```

There's a Migration Guide as well: [Guide](https://github.com/pytorch/text/blob/master/examples/legacy_tutorial/migration_tutorial.ipynb)

