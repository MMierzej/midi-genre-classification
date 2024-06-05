# MIDI genre classification

A final project for Machine Learning at University of Wrocław.

A collaborative effort of:
- [@Kristopher38](https://github.com/Kristopher38)
- [@gimboleo](https://github.com/gimboleo)
- [@MMierzej](https://github.com/MMierzej) (myself)

## Code

Refer to the [MIDI_genre_classification.ipynb](./MIDI_genre_classification.ipynb) for the code and its execution results.

## Brief introduction

The aim of the project is to classify pieces of music to one of the genres. The representation of music we are working with is MIDI, a format which encodes music as a series of chronological events (such as playing a certain note), separated into tracks and channels which correspond to instruments. From these MIDIs we extract features relevant to distinguishing the genre, and we apply classificators which work with that gathered information.

## A bit of details

### Dataset

The dataset we prepared for this project is based on: the [Lakh](https://colinraffel.com/projects/lmd/) dataset which contains MIDIs of songs, the [Million Song Dataset](http://millionsongdataset.com/) which establishes quasiuniversal (meaning respected by several other datasets) IDs of pieces of music (along with providing metadata about them), the [MSD Tagtraum](https://www.tagtraum.com/msd_genre_datasets.html) and the [Acousticbrainz Genre](https://mtg.github.io/acousticbrainz-genre-dataset/) datasets which provide genres for IDs from the Million Song Dataset. These sets were combined so that MIDIs are matched to the ID and the genre of the song each of them encodes.

The actual working (filtered) dataset consists of ~1200 MIDIs, ~170 per each of the 7 genres: Electronic, Metal, RnB, Country, Jazz, Rock, Pop. The proportions train:test:valid is 60:20:20.

### Features

In order to predict the genre of a song it's necessary to extract some features from the MIDI. Some music traits we decided to capture are:
- pitch
- velocity of notes (their "intensity")
- duration of and space between notes
- polyphony (how many notes are being played simultaneously)
- used instruments
- melodic intervals (musical distance between consecutive notes)
- shape of melodies
- time signature
- tempo
- balance of registers (low, mid, high)

After extracting features for all the used tracks, we analyzed entropies of columns, which gave us a rough idea about the importance of each feature. However, it was not as straightforward to just use the columns with the highest amount of contained information and leave out the rest. To guide the choice of feature subset to be used in the classification, we ran a genetic algorithm whose target function was the classifier output and the evaluated entities were bit encodings of subsets of all features.

As an implementation note, some feature extractors we implemented ourselves, however the more non-trivial ones were extracted using the [music21](https://github.com/cuthbertLab/music21) library.

### Used classifiers

We tried K-nearest neighbors, logistic regression, support vector machine, and neural networks. The numbers will be discussed in the *Conclusion* section.

## Conclusion

### Results

The above-mentioned genetic algorithm helped us choose the following features to use in classification:
- pitch_avg
- pitch_std
- pitch_min
- pitch_max
- velocity_avg
- velocity_std
- polyphony_avg
- polyphony_std
- polyphony_max
- Brass_Fraction
- Electric_Instrument_Fraction
- Violin_Fraction
- Melodic_Interval_Histogram_2
- Melodic_Interval_Histogram_3
- Melodic_Interval_Histogram_5
- Melodic_Interval_Histogram_7
- Melodic_Interval_Histogram_9
- Melodic_Interval_Histogram_11
- Melodic_Interval_Histogram_12
- Melodic_Interval_Histogram_14
- Melodic_Interval_Histogram_15
- Melodic_Interval_Histogram_16
- Melodic_Interval_Histogram_21
- Melodic_Interval_Histogram_22
- Melodic_Interval_Histogram_23
- Melodic_Interval_Histogram_24
- Melodic_Interval_Histogram_25
- Melodic_Interval_Histogram_26
- Melodic_Interval_Histogram_27
- Melodic_Interval_Histogram_28
- Melodic_Interval_Histogram_29
- Melodic_Interval_Histogram_32
- Melodic_Interval_Histogram_35
- Melodic_Interval_Histogram_37
- Melodic_Interval_Histogram_40
- Melodic_Interval_Histogram_44
- Melodic_Interval_Histogram_45
- Melodic_Interval_Histogram_47
- Melodic_Interval_Histogram_48
- Pitch_Class_Distribution_0
- Pitch_Class_Distribution_2
- Pitch_Class_Distribution_3
- Pitch_Class_Distribution_8
- Pitch_Class_Distribution_11
- Direction_of_Motion
- Repeated_Notes
- Importance_of_Bass_Register
- Importance_of_Middle_Register
- Duration_of_Melodic_Arcs
- tempo
- resolution
- ts_numerator

based on the classifiers' performance on the test part of the dataset.

![Progression of GA](https://i.imgur.com/NMX2mJW.png)

After choosing the features, we ran classifiers on the validation dataset using those features, and obtained the following results:

|     |        |
|-----|--------|
| SVM | 34.87% |
| KNN | 39.92% |
| LR  | 33.61% |
| NNs | 31.51% |

where the SVM classifier was the scikit's SVC, the KNN used 26 nearest neighbors, the logistic regression classifier was scikit's default one, the neural network used was `MLPClassifier(alpha=1e-05, hidden_layer_sizes=(10, 10), random_state=1)`. Below are the confusion matrices for each classifier. Rows' labels are the true labes, and columns' labels are the predicted ones.

| SVM ||||||||
|------------|------------|-----|------|-----|------|-------|---------|
|            | electronic | pop | rock | rnb | jazz | metal | country |
| electronic | 15 |  1 |  7 |  3 |  4 |  3 |   1 |
| pop        | 17 |  4 |  5 |  5 |  0 |  1 |   2 |
| rock       |  8 |  5 |  4 |  5 |  2 |  4 |   6 |
| rnb        |  8 |  6 |  4 | 12 |  2 |  0 |   2 |
| jazz       |  9 |  5 |  0 |  6 | 12 |  0 |   2 |
| metal      |  8 |  1 |  2 |  0 |  1 | 22 |   0 |
| country    |  5 |  5 |  3 |  6 |  1 |  0 |  14 |

<br />

| KNN ||||||||
|------------|------------|-----|------|-----|------|-------|---------|
|            | electronic | pop | rock | rnb | jazz | metal | country |
| electronic | 9 | 2 | 3 | 1 | 7 | 8 | 4|
| pop        | 5 | 3 | 5 | 4 | 4 | 2 |11|
| rock       | 0 | 4 | 6 | 5 | 3 | 7 | 9|
| rnb        | 1 | 6 | 4 | 8 | 5 | 1 | 9|
| jazz       | 0 | 0 | 1 | 6 |21 | 2 | 4|
| metal      | 1 | 0 | 1 | 2 | 3 |26 | 1|
| country    | 0 | 3 | 4 | 3 | 2 | 0 |22|  

<br />

| LR ||||||||
|------------|------------|-----|------|-----|------|-------|---------|
|            | electronic | pop | rock | rnb | jazz | metal | country |
| electronic | 19 | 0 | 4 | 0 |  5 |  4 |  2 |
| pop        | 22 | 2 | 3 | 5 |  0 |  1 |  1 |
| rock       | 10 | 5 | 4 | 4 |  3 |  5 |  3 |
| rnb        | 14 | 4 | 4 | 9 |  2 |  0 |  1 |
| jazz       |  9 | 3 | 1 | 6 | 14 |  0 |  1 |
| metal      |  8 | 0 | 2 | 0 |  2 | 22 |  0 |
| country    | 11 | 5 | 5 | 1 |  2 |  0 | 10 |  

<br />

| NNs ||||||||
|------------|------------|-----|------|-----|------|-------|---------|
|            | electronic | pop | rock | rnb | jazz | metal | country |
| electronic |  9 |  1 |   1|   3 |   8 |    9 |  3 |
| pop        | 14 | 1  |  2 |  3  |  5  |   4  |  5 |
| rock       |  6 | 2  |  2 |  4  |  5  |   7  |  8 |
| rnb        | 12 | 2  |  4 |  3  |  7  |   1  |  5 |
| jazz       |  1 | 1  |  1 |  7  | 20  |   3  |  1 |
| metal      |  3 | 0  |  1 |  0  |  2  |  28  |  0 |
| country    |  6 | 4  |  3 |  2  |  6  |   1  | 12 |

### Lessons learned

- It seems that tembre (what the music actually sounds like), among other audio features, is a significant factor that makes genres distinguishable. MIDI doesn't really carry that information, besides maybe the instruments used.
- Probably, the features we extracted don't possess enough power to attain outstanding accuracy with the dataset being only MIDIs. There are many possible features which leverage some (possibly large) degree of music theory, however they are very tricky to extract.
- There might be patterns characteristic for certain genres, which are not detectable in a dataset that small.
- It's quite a challenge to prepare the data.
