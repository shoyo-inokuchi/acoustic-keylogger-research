#  Acoustic Keylogging Attack - Research

[![CircleCI](https://circleci.com/gh/shoyo/acoustic-keylogger/tree/master.svg?style=shield)](https://circleci.com/gh/shoyo/acoustic-keylogger/tree/master)

## Overview
A *keyboard acoustic emanations attack* is a form of a [side-channel
attack](https://en.wikipedia.org/wiki/Side-channel_attack) where an attacker
extracts what a victim typed on his or her keyboard using just the audio signal
of the typing. Such a keylogging attack can result in passwords and
confidential information being stolen from just a compromised microphone, and
thus has severe security implications.

During my time in university, I explored whether many of the techniques that
have become de facto in the fields of voice recognition and natural language
processing can translate over to an implementation of an audio-based keylogging
attack. With the advent of several open-source machine learning libraries in
the past decade, I wondered whether such an attack is becoming increasingly
accessible to implement and should receive more awareness from the general
public.

I've since graduated from university and am no longer actively researching
this topic. However, details about the results and methodology are described below
for fellow researchers that are interested in investigating this topic further.

### Objective
Evaluate the threat of a *keyboard acoustic emanations attack* in the current
machine learning landscape by creating a proof-of-concept pipeline for
executing such an attack and measuring its __accuracy__, __practicality__, and
__accessibility__.
* __Accuracy__: How well does the pipeline approximate typed keys?
* __Practicality__: How robust is the pipeline under realistic conditions?
* __Accessibility__: How easily can the pipeline be built? (with regards to
  prerequisite knowledge and used technology)

### Extent of Research
An essential component of the pipeline is the ability to distinguish between
key sounds emitted by each key on a keyboard. There must be quantitative evidence
that keys (or groups of keys) emit different sounds, and they do so consistently
under regular conditions. 

For certain keyboards and typing patterns, the results suggest that key sounds
emitted can indeed be clustered by the position of the key on the keyboard. Each point
represents a single keystroke's 13,230-length audio vector (0.3s slice sampled at 44,100Hz)
mapped to a 2-D vector space.

![mx-brown tSNE clusters](https://github.com/shoyo-inokuchi/acoustic-keylogger-research/blob/master/lab/figs/vp3-brown.png)
*t-SNE clusters formed by keystroke sounds generated by a VP3 mechanical keyboard
with Cherry MX Brown switches*

![apple-butterfly tSNE clusters](https://github.com/shoyo-inokuchi/acoustic-keylogger-research/blob/master/lab/figs/macbook2016.png)
*t-SNE clusters formed by keystroke sounds generated by a Macbook Pro 2016 with
Apple butterfly switches*

### Methodology
The results above were produced by processing audio data of *non-overlapping* keystrokes
(i.e. the key-up of each keystroke occurs before the key-down of the next keystroke). Each
keystroke is extracted into a [mel-frequency ceptral coefficients](https://en.wikipedia.org/wiki/Mel-frequency_cepstrum)
feature vector and embedded with [t-SNE](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding).
Note that the points above are colored and labeled to visualize the accuracy of the clustering. In
practice, the attacker would need to perform further processing (cluster labeling) after this step
which may or may not accurately predict which key belongs to which cluster.

The functions used to process the data are located in the `acoustic_keylogger` package. I don't
have any immediate plans to write external documentation on how to use the package, but each
function in it has relatively detailed docstrings so please refer to them if you'd like to use
`acoustic_keylogger` in your own research.

Cluster labeling involves taking as input a sequence of "cluster IDs" and assigning a key type
to each cluster ID. If we can make assumptions about the content that was being typed (such as the
language or topic) then we can treat this as a time-series prediction problem to be solved with
Hidden Markov Chains or recurrent neural networks.

### The Pipeline
This is the pipeline that was being implemented when this project was actively developed.

* (Done) __Data Collection__ - Gathering a diverse dataset of typing sounds recorded
under realistic conditions

* (Done) __Keystroke Detection__ - Identifying all of the keystroke sounds in a given
audio file

* (Done) __Keystroke Feature Extraction__ - Preprocessing each keystroke sound for
further analysis

* (Done) __Clustering__ - Forming clusters with the preprocessed keystroke data

* __Predictive Cluster Labeling__ - Identifying which clusters correspond to
which key type

* __Iterative Pseudo-labeled Supervised Training__ - Training a classifier
using the predicted labels and iterating

This pipeline is modeled after the research described in [*Keyboard Acoustic Emanations Revisited* by L. Zhuang, F. Zhou, J. D. Tygar in 2005](https://www.cs.cornell.edu/~shmat/courses/cs6431/zhuang.pdf). I highly recommend reading this paper for those that want to explore this field of research.

## Setting up
### Option 0 - Using your own research environment
I'd assume many readers of this repository already have their own environment for conducting numerical
research (with Jupyter, NumPy etc.).

If you'd like to tinker around in your own environment, simply copy the `acoustic_keylogger` package into
your own machine, (possibly) update your `PYTHONPATH`, and import the functions as needed.

### Option 1 - Docker
This project uses a Python development environment and a PostgreSQL database to
manage various audio data. I chose Postgres due to variable-length array support,
but feel free to edit the config to use your preferred database. This option
spins up the Jupyter environment and database with Docker Compose.  

* Install [Docker](https://www.docker.com/products/docker-desktop).  
* Build images with `$ docker-compose build`. This is only required when dependencies or Docker config are updated.

This step will install all dependencies for `env` (such as Jupyter, Tensorflow,
NumPy etc.) and mount your local file system with the file system within the
`env` Docker container.

* Spin up the database and development environment with `$ docker-compose up`.

This should open up the database for connections and connect __http://localhost:8888__ to the Jupyter notebook.

### Option 2 - No Docker
Docker requires more overhead memory and comes with little quirks in the development environment
with the current setup (like having to manually open the Jupyter notebook). I
find that a lot of times using Docker for small tweaks is a bit overkill, so
I'm leaving this option here.

* Install the latest version of Python that's compatible with the dependencies (currently 3.8 due to Tensorflow 2.4.1).
To downgrade Python without overriding your current version, you can install [conda](https://www.anaconda.com/distribution/)
and run

        $ conda install python=<version>

* Set up a Python virtual environment. You can use conda, pipenv, virtualenvwrapper etc. for managing
multiple environments.

* Install dependencies with

        $ pip install -r requirements.txt  

* Make sure Python can find the `acoustic_keylogger` package with

        $ export PYTHONPATH=/path/to/repo/acoustic-keylogger/

  and can connect to the test database with

        $ export TEST_DATABASE_URL=postgresql+psycopg2://postgres@acoustic-keylogger_db_1:5432

  You can add these commands to your configuration file (`~/.bashrc`, `~/.zshrc`, etc.) so it gets loaded
  between terminal sessions.

* Open Jupyter notebook with

        $ jupyter notebook


This option can be simpler if you're unfamiliar with Docker or you don't need
to access the database. (Though the latter should still be possible using local
postgres commands)


## Testing [![CircleCI](https://circleci.com/gh/shoyo/acoustic-keylogger/tree/master.svg?style=shield)](https://circleci.com/gh/shoyo/acoustic-keylogger/tree/master)

Tests are implemented for the __acoustic_keylogger__ package, which contains
various functions for audio processing and data management.
These tests are contained in __tests/test_acoustic_keylogger__.

To run tests with the Docker configuration (Option 1), execute:

    $ docker-compose run env pytest -q tests

To run tests with no Docker configuration (Option 2), execute:

    $ python -m pytest -q tests

__Note:__ Both of the commands above are assumed to be executed from the root
directory of this repository.


## Relevant Research Papers
Many research papers were published in the mid-2000s concerning the topic of
keyboard acoustic emanations attacks. Some research, such as [*Keyboard
Acoustic Emanations Revisited* by L. Zhuang, F. Zhou, J. D. Tygar in
2005](https://www.cs.cornell.edu/~shmat/courses/cs6431/zhuang.pdf),
demonstrated extremely accurate results (96% chars recovered from 10 minute
sound recording) even without labeled training data.

### Supervised Methods
  * [*Keyboard acoustic emanations*](https://ieeexplore.ieee.org/document/1301311)
    by D. Asonov, R. Agrawal. 2004.

### Unsupervised Methods
  * [*Keyboard Acoustic Emanations Revisited*](https://www.cs.cornell.edu/~shmat/courses/cs6431/zhuang.pdf)
  by L. Zhuang, F. Zhou, J. D. Tygar. 2005.
  * [*Dictionary Attacks Using Keyboard Acoustic Emanations*](https://www.eng.tau.ac.il/~yash/p245-berger.pdf)
  by Y. Berger, A. Wool, A. Yeredor. 2006.
