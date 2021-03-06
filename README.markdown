[![Build Status](https://travis-ci.org/ckan/deadoralive.svg)](https://travis-ci.org/ckan/deadoralive)
[![Coverage Status](https://img.shields.io/coveralls/ckan/deadoralive.svg)](https://coveralls.io/r/ckan/deadoralive)
[![Latest Version](https://img.shields.io/pypi/v/deadoralive.svg)](https://pypi.python.org/pypi/deadoralive/)
[![Downloads](https://img.shields.io/pypi/dm/deadoralive.svg)](https://pypi.python.org/pypi/deadoralive/)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/deadoralive.svg)](https://pypi.python.org/pypi/deadoralive/)
[![Development Status](https://img.shields.io/pypi/status/deadoralive.svg)](https://pypi.python.org/pypi/deadoralive/)
[![License](https://img.shields.io/pypi/l/deadoralive.svg)](https://pypi.python.org/pypi/deadoralive/)


Dead or Alive
=============

Dead or Alive is a simple dead link checker service: it's a command-line script
or cron job that checks your site for broken links and posts the results back
to your site using an API (defined below) that your site must implement.

Design: [seanh.cc/posts/background-tasks-as-simple-web-services](http://seanh.cc/posts/background-tasks-as-simple-web-services/)

For [CKAN](http://ckan.org/) sites you can install
[ckanext-nextgeossdeadoralive](https://github.com/NextGeoss/ckanext-nextgeossdeadoralive) to add Dead
or Alive support.

In the future we'd like to make this into a web service rather than just a
cron job.  This will enable _ad-hoc_ link checking in response to user
interactions (i.e. the user clicks on a "check this link/these links now"
button on the client website, or checking a new resource as soon as a user
creates it) as well as the currently implemented automatic hourly checks.
See <https://github.com/ckan/deadoralive/issues/1>.


Requirements
------------

Python 2.6 or 2.7.


Installation
------------

To install run:

    pip install -e git+https://github.com/NextGeoss/nextgeoss-deadoralive#egg=nextgeoss-deadoralive
    python setup.py develop
    pip install -r dev-requirements.txt

To install in a docker, put this in the dockerfile:

    RUN pip install -e git+https://github.com/NextGeoss/nextgeoss-deadoralive@v0.1.4#egg=nextgeoss-deadoralive
    RUN pip install -r https://raw.githubusercontent.com/NextGeoss/nextgeoss-deadoralive/master/dev-requirements.txt

If you want to check a CKAN site for broken links you also need to install
[ckanext-deadoralive](https://github.com/NextGeoss/ckanext-nextgeossdeadoralive) on your
CKAN site.

To install for development, create and activate a Python virtual environment
then do:

    git clone https://github.com/NextGeoss/nextgeoss-deadoralive.git
    cd deadoralive
    python setup.py develop
    pip install -r dev-requirements.txt


Usage
-----

To check a site for broken links run this in the commandline:

    deadoralive --org <org_to_check> --user <user_org> --pasw <pasw_org> --url <your_site> --apikey <your_api_key>

Replace `<your_site>` with the URL of the CKAN or other client
site you want to check (e.g. `http://demo.ckan.org`)(Required).

Replace `<your_api_key>` with the API key of the site user that you want the
link checker to run as, you can get it from your ckan site in the user logged page (Required).

Replace `<org_to_check>` with the organization to check for (Not required, if it is empty, the client checks all the organizations).

Replace `<user_org>` with the user you have created for the 
organization to check (Not required).

Replace `<pasw_org>` with the pasword for the user of the 
organization to check (Not required).

Remember to create a user for CMEMS and SciHub in order to give authentication to the link checker,
but if the link checker is not going to test against one or none them, you do not need to put those parameters.

In the ckanext-nextgeossdeadoralive you can change the filter to modify which organizations
should be filtered [ckanext-nextgeossdeadoralive's config.py](https://github.com/NextGeoss/ckanext-nextgeossdeadoralive/blob/master/ckanext/deadoralive/config.py) 
in organization_to_filter variable.

If the site is a CKAN site the API key should be the CKAN API key of one of the
users listed in the `ckanext.deadoralive.authorized_users` config setting in
your CKAN config file (see
[ckanext-nextgeossdeadoralive's docs](https://github.com/NextGeoss/ckanext-nextgeossdeadoralive) for
details).


Adding a Cron Job
-----------------

To setup the link checker to run automatically you can add a cron job for it.
On most UNIX systems you can add a cron job by running `crontab -e` to edit
your crontab file. Add a line like the following to the file and save it:

    @hourly deadoralive (add here the credentials specified above) --url '<your_site>' --apikey <your_api_key>

As before replace `<your_site>` with the URL of the site you want to check,
and `<your_api_key>` with an API key from the site.

You can also use `@daily` or `@weekly` instead of `@hourly` if you want link
checking to happen less often.


Running the Link Checker on a Different Machine
-----------------------------------------------

Although the ckanext-deadoralive extension has to be installed and activated on
the CKAN site that you want to check the links of, the link checker script
itself does not need to be run from the same machine. Because it does all
communication with the CKAN site via CKAN's API, you can run it from a
different server or from your laptop: just install and run deadoralive
as described above.


Running Multiple Link Checkers at Once
--------------------------------------

It's perfectly fine to run multiple instances of the `deadoralive` link
checker script at once. For example:

* Have a cron job setup on the server to run the link checker hourly, but
  occassionally run another copy of the link checker manually on your laptop.

* Have multiple cron jobs on multiple servers running link checkers against
  the same CKAN site.

CKAN will hand out different resources to each link checker, and won't let two
checkers check the same resource at the same time.


### Running Multiple Link Checkers on the Same Machine

By default deadoralive uses a socket to prevent two instances of the
script from running at the same time on the same machine. This is to prevent
link checker processes from piling up when the link checker is being run as a
cron job and doesn't finish checking all the links before cron runs it again.

If you _want_ to run multiple instances on the same machine at the same time,
just use the `--port` option to specify a different port for each.
For example:

    deadoralive (add here the credentials specified above) --url '<url>' --apikey <apikey> --port 4567
    deadoralive (add here the credentials specified above) --url '<url>' --apikey <apikey> --port 4568
    deadoralive (add here the credentials specified above) --url '<url>' --apikey <apikey> --port 4569

(deadoralive doesn't actually do anything with the port, it just binds a
socket to it to prevent any other deadoralive processes with the same port
from running.)


Checking Multiple Sites
-----------------------

You can use a single instance of the link checker to check multiple sites:
just pass it different `--url` and `--apikey` arguments.

For example you might setup three cron jobs to check three different sites,
giving each job a different port so that they can run simultaneously:

    @hourly deadoralive (add here the credentials specified above) --url '<first_site>' --apikey <first_api_key> --port 4567
    @hourly deadoralive (add here the credentials specified above) --url '<second_site>' --apikey <second_api_key> --port 4568
    @hourly deadoralive (add here the credentials specified above) --url '<third_site>' --apikey <third_key> --port 4569


API
---

The API that a site must implement to be compatible with deadoralive is as
follows:

* All request and response bodies are in JSON-formatted text.

* deadoralive sends the API key (from its `--apikey` option) in all HTTP
  requests as an Authorization header.

  For example `deadoralive --org <org_to_check> --user <user_org> --pasw <pasw_org> --url <url> --apikey foo` will send requests with
  header `Authorization: foo`.

* `GET /deadoralive/get_resources_to_check`

  Return a list of IDs of resources to be checked. An ID can be any string that
  uniquely identifies a resource that has a URL.

  deadoralive will request this endpoint once at the start of each run.

  Example output: `["id_1", "id_2", ...]`.

* `GET /deadoralive/get_url_for_resource?resource_id=<resource_id>`

  Return the URL of the link to be checked for the given resource ID.

  deadoralive will request this endpoint once for each of the resource IDs
  it was given by the previous request.

  Example output: `{"url": "http://demo.ckan.org/data.csv"}`

* `POST /deadoralive/upsert`

  deadoralive will post to this endpoint to report the result of each link
  check that it does.

  Example post body:

      {"resource_id": "id_1",
       "alive": false,
       "status": 500,
       "reason": "Internal Server Error",
       "url": "http://demo.ckan.org/data.csv"
      }

See [ckanext-deadoralive](https://github.com/NextGeoss/ckanext-nextgeossdeadoralive) for a
reference implementation.


Running the Tests
-----------------

First activate your virtualenv then install the dev requirements:

    pip install -r dev-requirements.txt

Then to run the tests do:

    nosetests

To run the tests and produce a test coverage report do:

    nosetests --with-coverage --cover-inclusive --cover-erase --cover-tests
