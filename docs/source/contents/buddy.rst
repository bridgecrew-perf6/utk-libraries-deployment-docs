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

Current versions of the yamls are kept in a `private repository <https://github.com/utkdigitalinitiatives/utlibraries_deployment>`_.
Sanitized versions of the pipelines are also stored here for documentation sake.

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

Using this image, we build and run a container that:

1. Turns off the memory limit for PHP
2. Installs required Debian packages for PHP and composer
3. Downloads and installs compose
4. Configures and executes PHP extensions

These steps are reflected in the corresponding yaml:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 20-39

Once the container is built, composer is run inside of the utk-libraries git repository in the working directory:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 11-19
    :emphasize-lines: 3, 6-9

---------------------------------------
Step 2: Build React Apps & Sage 9 theme
---------------------------------------

Next, we build out the React components of the website. To do this, we use the :code:`node` docker image from docker hub
that is tagged with the :code:`10` tag.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 43-47
    :emphasize-lines: 4-5

We then setup the environment by installing gulp and grunt-cli.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 65-66

Finally, we build out the individual react applications and ultimately our theme based on sage:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 48-64

Our action also defines the react path as a variable:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 70-73
    :emphasize-lines: 3

---------------------------
Step 3: Set Release Version
---------------------------

Next, we build a container to build our release. To do this, we use the :code:`ubuntu` docker image from docker hub with
the :code:`18.04` tag.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 76-80
    :emphasize-lines: 4-5

We then install git in this new container.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 90-91

And ultimately tag our new release by taking our global variables used for semantic versioning and adding a 1 to the
patch number:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 81-89
    :emphasize-lines: 3

Like previous steps, all of this runs against our working directory that holds a copy of our github repository with the
code base:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 76-80
    :emphasize-lines: 3

------------------------------
Step 4: Push Release to Github
------------------------------

The following step is something that `Mark <https://github.com/markpbaggett>`_ believes should be reworked.

In this step, we push the tagged release we just built to GitHub, but oddly we do it in a new container that uses the
same image that we had in the previous step.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 97-101
    :emphasize-lines: 3-4

From a performance point of view, this definitely seems questionable.  Yes, this is only an issue when changes are made
or if you are building for the first time, but it still seems so wrong.  In `Mark's <https://github.com/markpbaggett>`_
the execute commands should be combined into the previous step.

The setup rules are exactly the same as the previous step.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 116-117

The steps that are executed simply add the SSH key defined by the `utk-libraries <https://github.com/utkdigitalinitiatives/utk-libraries>`_
deploy keys settings to the authorized_keys file in the user directory of this container so that the tagged released can
be pushed. We then create a fake git config file for Buddy to more easily see the changes that are made by Buddy rather
than an individual. Finally, we set our origin to our Github repo and push the tagged release.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 102-114

-------------------------------------------------------
Step 5 and 6: Find and Replace for our Sage-based theme
-------------------------------------------------------

This section describes 2 important steps that is not clear from reading the hashed yaml. It is critical that
both of these run so that our `Sage-based <https://roots.io/sage/>`_ theme builds.

In the first step, we open the :code:`.gitignore` file in our Sage-based theme directory and replace several lines so
that they are no longer ignored.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 123-134
    :emphasize-lines: 3

In the next step, we take similar actions versus the :code:`wp-content/mu-plugins/utk_library/assets/.gitignore`:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 135-140
    :emphasize-lines: 3

The reason for this is that Pantheon needs these files in order to build correctly.  This becomes more clear in the next
steps.

-------------------------------
Step 7: Push to Pantheon Master
-------------------------------

**NOTE**: In order to understand fully what's going on in this step, you need to import the yaml into a buddy pipeline.
If not, this will look like wizardry.

This step, takes the output of all preceding actions, commits them, and force pushes all files to the :code:`master`
branch in a git repository on Pantheon. The repository's location is highlighted below but obscured for security purposes:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 141-148
    :emphasize-lines: 5

As stated above, by looking at the raw yaml, there is a lot of magic here.  You can only fully appreciate what is
happening by importing this to a Buddy pipeline so that the action is decrypted. Notice, that there is an environmental
variable that is hashed here:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 141-148
    :emphasize-lines: 4

When you look at this in Buddy, this variable is decrypted to this:

.. code-block:: shell

    echo -e 'ssh-rsa ###REPLACED_HASH### utk-libraries-1 Key\n' >> ~/.ssh/authorized_keys
    chmod 0600 ~/.ssh/authorized_keys

So that :code:`env_key` includes the SSH key that allows Buddy to write to Pantheon.

It's not clear how this next part is hashed in the yaml, but there is a **CRITICAL** instruction also captured here that
relates to the find and replace steps. This too only becomes clear on import to Buddy:

.. image:: ../images/buddy_commit_all_changes.png
    :width: 600
    :Alt: Buddy rule that captures all changes to a fresh commit pre-push

Now we can start to better understand what is happening here. In this step, we are:

1. Adding an SSH key to Buddy that allows us to write to Pantheon
2. Committing all files that have been created or modified by previous actions (Including those files that were originally ignored)
3. Force pushing the output to the :code:`master` branch on Pantheon

--------------------------------
Step 8: Push to Pantheon Sandbox
--------------------------------

**NOTE**: In order to understand fully what's going on in this step, you need to import the yaml into a buddy pipeline.
If not, this will look like wizardry.

This step, takes the output of all preceding actions, commits them, and force pushes all files to the :code:`sandbox`
branch in a git repository on Pantheon. The repository's location is highlighted below but obscured for security purposes:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 149-157
    :emphasize-lines: 5

As stated above, by looking at the raw yaml, there is a lot of magic here.  You can only fully appreciate what is
happening by importing this to a Buddy pipeline so that the action is decrypted. Notice, that there is an environmental
variable that is hashed here:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 149-157
    :emphasize-lines: 4

When you look at this in Buddy, this variable is decrypted to this:

.. code-block:: shell

    echo -e 'ssh-rsa ###REPLACED_HASH### utk-libraries-1 Key\n' >> ~/.ssh/authorized_keys
    chmod 0600 ~/.ssh/authorized_keys

So that :code:`env_key` includes the SSH key that allows Buddy to write to Pantheon.

It's not clear how this next part is hashed in the yaml, but there is a **CRITICAL** instruction also captured here that
relates to the find and replace steps. This too only becomes clear on import to Buddy:

.. image:: ../images/buddy_commit_all_changes.png
    :width: 600
    :Alt: Buddy rule that captures all changes to a fresh commit pre-push

Now we can start to better understand what is happening here. In this step, we are:

1. Adding an SSH key to Buddy that allows us to write to Pantheon
2. Committing all files that have been created or modified by previous actions (Including those files that were originally ignored)
3. Force pushing the output to the :code:`sandbox` branch on Pantheon

=========
Rationale
=========

If you're reading this, you may be wondering: **why do we push a master and sandbox branch to Pantheon?**

The :code:`master` branch described in step 7 is the primary dev environment.  It has its own database and is what we
normally look at when we are testing.  If we pull the database from test to dev, it updates this.

The :code:`sandbox` branch described here exists because some stakeholders wanted a "development" area to try new things
and we needed a way to insure that this wasn't overwritten when the database was updated. As far as `Mark <https://github.com/markpbaggett>`_
knows, this was only ever used by Teaching and Learning.

-------------------------------
Step 9: Push to Github Upstream
-------------------------------

**NOTE**: Like the two preceding steps, in order to understand fully what's going on in this step, you need to import
the yaml into a buddy pipeline. If not, this will look like wizardry.

This step, takes the output of all preceding actions, commits them, and force pushes all files to the
`upstream repository <https://github.com/utkdigitalinitiatives/utk-libraries-upstream>`_ in Github.

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 158-165
    :emphasize-lines: 7

Like the two preceding steps, the yaml includes a hashed key that includes instructions for authorizing write access to
the Github repo and instructions to commit files from all preceding actions that are only readable once this yaml has
been imported into a Buddy pipeline:

.. code-block:: shell

    echo -e 'ssh-rsa ###REPLACED_HASH### utk-libraries-1 Key\n' >> ~/.ssh/authorized_keys
    chmod 0600 ~/.ssh/authorized_keys

.. image:: ../images/buddy_commit_all_changes.png
    :width: 600
    :Alt: Buddy rule that captures all changes to a fresh commit pre-push

This action makes some things clear:

1. Deployment to Pantheon happens in Buddy prior to deployment to Github
2. We keep a copy of things on Pantheon in Github for easy diffing between the original code repo and the deployment repo.

----------------------
Step 10: Apply Updates
----------------------

This last action is the reason why we get an error when we try to import this yaml in Buddy. The error message makes you
think the import fails, but actually it works except that this action doesn't come along.  You have to do that yourself.

This action calls an entirely different pipeline and forces it to run:

.. literalinclude:: ../sanitized_files/2022BuildTestandDeployMasterBranch.yml
    :language: yaml
    :lines: 166-171

This pipeline isnt' described in detail yet, but should be.

Essentially, the action builds a PHP environment so that Pantheon's `terminus-installer <https://github.com/pantheon-systems/terminus-installer>`_
can be installed.

Once installed, we login to the environment in Pantheon, clear cache on volumes, and apply updates to volumes.

.. code-block:: yaml

    execute_commands:
        - "terminus auth:login --machine-token=$machine_token"
        - ""
        - "terminus site:upstream:clear-cache utk-volumes"
        - "terminus upstream:updates:apply --updatedb --accept-upstream utk-volumes.dev"

`Mark <https://github.com/markpbaggett>`_ is still not sure why this is important.
