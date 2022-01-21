CI/CD Workflows with Buddy
==========================

About Buddy
-----------

`Buddy <https://app.buddy.works/utk-libraries>`_ is the CI/CD platform that is used to build releases and deploy them to
our live site on Pantheon.

This section describes in details the projects and pipelines that are used for this process.

Active Project
--------------

The :code:`2022 utk-libraries` project is the current project used in Buddy for deploying and updating our WordPress
instance.

The project is based on the `utk-libraries <https://github.com/utkdigitalinitiatives/utk-libraries>`_ Github repository.
A deploy key is defined in this repository in the :code:`Settings > Deploy Keys` section.  This key allows Buddy to read
and push releases to this repository.  Without this key, deployment will not work.  Also, we use deploy keys so that this
process is not tied to any one individual.

The :code:`Project Settings` section in Buddy generated and defines the key that is used above in GitHub.

Other keys may be defined in the :code:`Variables, Keys, Assets` section. If they are relevant, they will be defined
below. This section also defines some global environmental variables including:

1. :code:`$major`: contains the base number for semantic versioning.
2. :code:`$minor`: contains the second number for semantic versioning.
3. :code:`$patch`: the last digit of the semantic versioning number for releases.
4. :code:`$theme`: the path that stores the location of the theme directory.


Pipelines
---------

CI/CD Workflows in this project are controlled by its pipeline.  There are currently two which are used in tandem and
described below.

Because they are used in tandem, you **CANNOT** simply import one yml without the other.  If you do this, you'll need to
tie them together in the end.  That is also described below.

#########################################
2022 Build, Test and Deploy Master Branch
#########################################

This is the primary pipeline used for build and deploy. This pipeline is split into 10 separate steps.

----------------------
Step 1: Install / Test
----------------------

The first step of the pipeline is to build an environment where we can install and test our application with composer.
To do this, we use the :code:`php` docker image from docker hub that is tagged with the :code:`7.2-stretch` tag.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 11-15
    :emphasize-lines: 4-5

