===========
 DEXTERITY
===========

I'm following the Dexterity Developer Manual.

https://docs.plone.org/external/plone.app.dexterity/docs/index.html

Dexterity Developer's Manual
============================

Creating a package
------------------

The Dexterity docs talk about using ZopeSkel but that binary doesn't
exist. I'm trying mr.bob::

  ../bin/mrbob -O example.conference bobtemplates:plone_addon

In src/example/conference/configure.zcml there is no
<browser:resourceDirectory> section; there is <include
package=".browser" /> and a browser/ dir with its own
configure.zcml. Assuming this is OK.

(I'm a bit disappointed the docs aren't keeping up with the unified
installer layout.)

Success: I can see example.conference in the installer:
http://localhost:8080/Plone/prefs_install_products_form

TODO: try TTW Dexterity content type creation:
http://localhost:8080/Plone/@@dexterity-types

Schema-driven types
-------------------

Again, docs don't match installer: we don't have the paster
templates::

  $ cd src/example.conference/

  $ ../../bin/paster addcontent -l
  Command 'addcontent' not known (you may need to run setup.py egg_info)

What should we have done? (we can develop the types TTW then export!).

For now, create the files manually; Dexterity is supposed to reduce
boilerplate from Archetypes.

Structure is like src/example.conference/src/example/conference/
and under it:

  -rw-r--r--   __init__.py
  -rw-r--r--   __init__.pyc
  drwxr-xr-x   browser
  -rw-r--r--   configure.zcml
  -rw-r--r--   interfaces.py
  -rw-r--r--   interfaces.pyc
  drwxr-xr-x   locales
  drwxr-xr-x   profiles
  -rw-r--r--   setuphandlers.py
  -rw-r--r--   setuphandlers.pyc
  -rw-r--r--   testing.py
  drwxr-xr-x   tests

I don't know where these .py files should be put -- under `browser`?
Ah, the github code shows the structure, I never would have guessed;
.py files are under example/conference with the configure.zcml and
testing.py.

I added it through ZMI, then see two content types, which I added:
* example.conference.program
* example.conference.session

When I do, I see they have no fields. Also, shouldn't these be
subortinate to Conference type? which we don't have? At least we've
been able to create and install new types. Perhaps it's a matter of
now adding fields: we did define the schema by Interface, so why don't
we have fields?

The startup logs show what might be problems::

  CMFQuickInstallerTool Multiple extension profiles found for product example.conference. Used profile: example.conference:default
  Products.GenericSetup.tool Importing profile profile-example.conference:default with dependency strategy upgrade.
  Products.GenericSetup.tool Applying main profile profile-example.conference:default

I also see we now have our ``ttw.contenttype`` that I
defined... TTW. It's already installed, so add one. Oh my god, it's
full of stars. No, really, all the fields I defined are there, with a
nice form to fill. Strangely it has two Title and Summary, which I
think were already provided in the TTW designer. When I save, I get
the second Title and Summary, the first I filled are ignored. There
are a bunch of fields that I don't think I need -- machinery like
Search Terms, Preview, Limit, Item Count; Table Columns (for tabular
view).


OT: no zopeskel or paster
--------------------------

Maybe I should have been following their examples of specifying paths
like "../bin/paster" instead of activating the virtualenv: turns out
my `paster` was not coming from my virtualenvironment but from some
errantly installed /usr/local/bin/paster, so I removed that. Now I
certainly can't do "paster addcontent" but at least I know why it
didn't know about "addcontent".

I think I need to add `zopeskel` and `paster` with their various
templates to the develop buildout so I can follow the tutorials. And
when should I use `mr.bob` versus these tools? Added to develop.cfg::

  parts +=
      paster

  [paster]
  recipe = zc.recipe.egg
  eggs =
         PasteScript
         ZopeSkel
         zopeskel.dexterity
         ${instance:eggs}

and now I have `paster` and `zopeskel` in my `bin/`

Retry the zopeskel/paster content type creation
-----------------------------------------------

If I answer the default::

  Grok-Based? (True/False: Use grok conventions to simplify coding?) [True]:

the setup.py has slightly different requirement::

  'plone.app.dexterity [grok]',

instead of::

  'plone.app.dexterity',

So re-do answering False to Grok, and now we get the non [grok] dexterity.

Now edit buildout.cfg to include `example.conference` then rerun buildout.


Schema-driven types (try again)
-------------------------------

Great, I'm still missing something::

  ../../bin/paster addcontent -l
  Warning: could not load entry point dexterity_behavior (ImportError: No module named dexterity.localcommands.dexterity)
  Warning: could not load entry point dexterity_content (ImportError: No module named dexterity.localcommands.dexterity)
  Available templates:
    No template

Smcmahon said: It is getting increasingly hard to keep zopeskel
working with new versions of setuptools.

Encolpe said: there may be a problem with zopeskel version pinning.

Djowett said:

  I've had fights like this in the past with zopeskel / paster.

  Nowadays though I avoid it.... choosing to use:

  mrbob for templating eggs
  creating dexterity either through the web - see plone training
  OR directly in code without templating - as there's a lot less
  boilerplate code in dexterity cf archetypes

I see the buildout of my newly added, unpinned zopeskel is 2.21.2::

  [versions]
  Cheetah = 2.2.1
  ZopeSkel = 2.21.2
  zopeskel.dexterity = 1.5.4.1

which Encolpe says I should have.



Model-driven types (XML)
------------------------

