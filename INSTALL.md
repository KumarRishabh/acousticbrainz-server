Installing AcousticBrainz Server
================================

AcousticBrainz Server consists of two core components: a web server that powers
acousticbrainz.org website and a number of background processes which perform
tasks for the website:

 * A high-level data extractor that processes incoming low-level information about recordings.
 * A tool to build models based on datasets created on the website

## Prerequisites

* [Python](https://www.python.org/) 2.7.x
* [PostgreSQL](http://www.postgresql.org/) >=9.4 (needs the JSONB data type)
* [Node.js](https://nodejs.org/en/) >6.0
* [Redis](https://redis.io)
* [pxz](http://manpages.ubuntu.com/manpages/trusty/man1/pxz.1.html) for
exporting and importing the data

For example in the latest Ubuntu, this command will install pre-requisites:

    $ sudo apt-get install python-dev python-virtualenv pxz redis-server \
        postgresql-9.4 postgresql-client-9.4 postgresql-server-dev-9.4

See https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions for instructions on how to install a recent version of node.js

## Web Server

It is recommended, although optional, to first set up a virtual environment and
activate it:

    $ virtualenv venv
    $ . ./venv/bin/activate

### Python dependencies

Then use `pip` to install the required Python dependencies:

    $ pip install -r requirements_development.txt

### Configuration

Copy `custom_config.py.example` to `custom_config.py` and edit its content to fit your environment.

You will need to add configuration options for `DATASET_DIR` and
`FILE_STORAGE_DIR` to point to directories that the user that you run the
server as can write to.

### Creating the database

After you tweak configuration file, the database needs to be created.

Make sure that you have a postgres superuser which can create users and databases.
If this user must be authenticated, change `POSTGRES_ADMIN_URI` and
`POSTGRES_ADMIN_AB_URI` in your `custom_config.py`. Then, to create the database run:

    $ python manage.py init_db

### Node.js dependencies

Node.js dependencies are managed using `npm`. To install these dependencies run:

    $ npm install

### Configuring and running the server

Continue from the configuration section in README.md

## High-level Data Extractor

To run high-level data extractor you'll need two things:

1. `streaming_extractor_music_svm` which is a part of the [Essentia library](http://essentia.upf.edu/)
2. *svm_models* which are available at http://essentia.upf.edu/documentation/svm_models/

### Building `streaming_extractor_music_svm`

To build the extractor binary you'll need to get [Gaia](https://github.com/MTG/gaia)
and [Essentia](https://github.com/MTG/essentia) libraries. Generally, you should
use installation instructions provided with them, but to save you some time
here's a set of steps that you can follow:

1. `git clone` Gaia and install it without using additional options during the
configuration step

2. `git clone` Essentia and configure it like this:

        $ ./waf configure --mode=release --with-gaia --with-example=streaming_extractor_music_svm

3. Build Essentia and copy `streaming_extractor_music_svm` binary from *build/src/examples/*
directory into *./hl_extractor/* in the project root.

### Installing SVM models

Download archive from http://essentia.upf.edu/documentation/svm_models/, extract
it, and move contents of the *svm_models* directory into
*/hl_extractor/svm_models* in the project root.

In the file `profile.conf.in`, you will need to change the paths of the
items in the `svm_models` list. If you make these paths relative, they must
be relative to the root directory of the project.

### Running

Run the high-level extractor using the `worker_manage.py` script:

    python worker_manage.py hl_extractor

## Dataset evaluator

Run it like this:

    python worker_manage.py dataset_evaluator

## Testing

Database configuration for testing is configured in `test_config.py`.
You will need to manually create a user and database and
set `SQLALCHEMY_DATABASE_URI` to use these values. The tests
will automatically build the database structure when running.