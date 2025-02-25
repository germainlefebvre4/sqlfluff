.. _releasenotes:

Release Notes
=============

This page aims to act as a guide for migrating between major SQLFluff
releases. Necessarily this means that bugfix releases, or releases
requiring no change for the user are not mentioned. For full details
of each individual release, see the detailed changelog_.

.. _changelog: https://github.com/sqlfluff/sqlfluff/blob/main/CHANGELOG.md

.. _upgrading_2_0:


Upgrading from 1.x to 2.0
-------------------------

Upgrading to 2.0 brings several important breaking changes:

* All bundled rules have been recoded, both from generic :code:`L00X` formats
  into groups within similar codes (e.g. an *aliasing* group with codes
  of the format :code:`AL0X`), but also given *names* to allow much clearer
  referencing (e.g. :code:`aliasing.column`).
* :ref:`ruleconfig` now uses the rule *name* rather than the rule *code* to
  specify the section. Any unrecognised references in config files (whether
  they are references which *do* match existing rules by code or alias, or
  whether the match no rules at all) will raise warnings at runtime.
* A complete re-write of layout and whitespace handling rules (see
  :ref:`layoutref`), and with that a change in how layout is configured
  (see :ref:`layoutconfig`) and the combination of some rules that were
  previously separate. One example of this is that the legacy rules
  :code:`L001`, :code:`L005`, :code:`L006`, :code:`L008`, :code:`L023`,
  :code:`L024`, :code:`L039`, :code:`L048` & :code:`L071` have been combined
  simply into :class:`LT01 <sqlfluff.rules.sphinx.Rule_LT01>`.

Recommended upgrade steps
^^^^^^^^^^^^^^^^^^^^^^^^^

To upgrade smoothly between versions, we recommend the following sequence:

#. The upgrade path will be simpler if you have a slimmer configuration file.
   Before upgrading, consider removing any sections from your configuration
   file (often :code:`.sqlfluff`, see :ref:`config`) which match the current
   :ref:`defaultconfig`. There is no need to respecify defaults in your local
   config if they are not different to the stock config.

#. In a local (or other *non-production*) environment, upgrade to SQLFluff
   2.0.x. We recommend using a `compatible release`_ specifier such
   as :code:`~=2.0.0`, to ensure any minor bugfix releases are automatically
   included.

#. Examine your configuration file (as mentioned above), and evaluate how
   rules are currently specified. We recommend primarily using *either*
   :code:`rules` *or* :code:`exclude_rules` rather than both, as detailed
   in :ref:`ruleselection`. Using either the :code:`sqlfluff rules` CLI
   command or the online :ref:`ruleref`, replace *all references* to legacy
   rule codes (i.e. codes of the form :code:`L0XX`). Specifically:

   * In the :code:`rules` and :code:`exclude_rules` config values. Here,
     consider using group specifiers or names to make your config simpler
     to read and understand (e.g. :code:`capitalisation`, is much more
     understandable than :code:`CP01,CP02,CP03,CP04,CP05`, but the two
     specifiers will have the same effect). Note that while legacy codes
     *will still be understood* here (because they remain valid as aliases
     for those rules) - you may find that some rules no longer exist in
     isolation and so these references may be misleading. e.g. :code:`L005`
     is now an alias for
     :class:`LT01, layout.spacing <sqlfluff.rules.sphinx.Rule_LT01>` but
     that rule is much more broad ranging than the original scope of
     :code:`L005`, which was only spacing around commas.

   * In :ref:`ruleconfig`. In particular here, legacy references to rule
     codes are *no longer valid*, will raise warnings, and until resolved,
     the configuration in those sections will be ignored. The new section
     references should include the rule *name* (e.g.
     :code:`[sqlfluff:rules:capitalisation.keywords]` rather than
     :code:`[sqlfluff:rules:L010]`). This switch is designed to make
     configuration files more readable, but we cannot support backward
     compatibility here without also having to resolve the potential
     ambiguity of the scenario where both *code-based* and *name-based*
     are both used.

   * Review the :ref:`layoutconfig` documentation, and check whether any
     indentation or layout configuration should be revised.

#. Check your project for :ref:`in_file_config` which refer to rule codes.
   Alter these in the same manner as described above for configuration files.

#. Test linting your project for unexpected linting issues. Where found,
   consider whether to use :code:`sqlfluff fix` to repair them in bulk,
   or (if you disagree with the changes) consider changing which rules
   you enable or their configuration accordingly. In particular you may notice:

   * The indentation rule (:code:`L003` as was, now
     :class:`LT02, layout.indent <sqlfluff.rules.sphinx.Rule_LT02>`) has had
     a significant rewrite, and while much more flexible and accurate, it is
     also more specific. Note that :ref:`hangingindents` are no longer
     supported, and that while not enabled by default, many users may find
     the enabling :ref:`implicitindents` fits their organisation's style
     better.

   * The spacing rule (
     :class:`LT01, layout.spacing <sqlfluff.rules.sphinx.Rule_LT01>`) has a
     much wider scope, and so may pick up spacing issues that were not
     previously enforced. If you disagree with any of these, you can
     override the :code:`sqlfluff:layout` sections of the config with
     different (or just more liberal settings, like :code:`any`).

.. _`compatible release`: https://peps.python.org/pep-0440/#compatible-release


Example 2.0 config
^^^^^^^^^^^^^^^^^^

To illustrate the points above, this is an illustrative example config
for a 2.0 compatible project. Note that the config is fairly brief and
sets only the values which differ from the default config.

.. code-block:: cfg

    [sqlfluff]
    dialect = snowflake
    templater = dbt
    max_line_length = 120

    # Exclude some specific rules based on a mixture of codes and names
    exclude_rules = RF02, RF03, RF04, ST06, ST07, AM05, AM06, convention.left_join, layout.select_targets

    [sqlfluff:indentation]
    # Enabling implicit indents for this project.
    # See https://docs.sqlfluff.com/en/stable/layout.html#configuring-indent-locations
    allow_implicit_indents = True

    # Add a few specific rule configurations, referenced by the rule names
    # and not by the rule codes.
    [sqlfluff:rules:capitalisation.keywords]
    capitalisation_policy = lower

    [sqlfluff:rules:capitalisation.identifiers]
    capitalisation_policy = lower

    [sqlfluff:rules:capitalisation.functions]
    extended_capitalisation_policy = lower

    # An example of setting a custom layout specification which
    # is more lenient than default config.
    [sqlfluff:layout:type:set_operator]
    line_position = alone


Upgrading to 1.4
----------------

This release brings several internal changes, and acts as a prelude
to 2.0.0. In particular, the following config values have changed:

* :code:`sqlfluff:rules:L007:operator_new_lines`` has been changed to
  :code:`sqlfluff:layout:type:binary_operator:line_position`.
* :code:`sqlfluff:rules:comma_style`` and
  :code:`sqlfluff:rules:L019:comma_style` have both been consolidated
  into :code:`sqlfluff:layout:type:comma:line_position`.

If any of these values have been set in your config, they will be
automatically translated to the new values at runtime, and a warning
will be shown. To silence the warning, update your config file to the
new values. For more details on configuring layout see :ref:`layoutconfig`.


Upgrading to 1.3
----------------

This release brings several potentially breaking changes to the underlying
parse tree. For users of the cli tool in a linting context you should notice
no change. If however your application relies on the structure of the SQLFluff
parse tree or the naming of certain elements within the yaml format, then
this may not be a drop-in replacement. Specifically:

* The addition of a new :code:`end_of_file`` meta segment at the end of
  the parse structure.
* The addition of a :code:`template_loop`` meta segment to signify a jump
  backward in the source file within a loop structure (e.g. a jinja
  :code:`for`` loop).
* Much more specific types on some raw segments, in particular
  :code:`identifier` and :code:`literal` type segments will now appear
  in the parse tree with their more specific type (which used to be called
  :code:`name`) e.g. :code:`naked_identifier`, :code:`quoted_identifier`,
  :code:`numeric_literal` etc...

If using the python api, the *parent* type (such as :code:`identifier`)
will still register if you call :code:`.is_type("identifier")`, as this
function checks all inherited types. However the eventual type returned
by :code:`.get_type()`` will now be (in most cases) what used to be
accessible at :code:`.name`. The :code:`name` attribute will be deprecated
in a future release.


Upgrading to 1.2
----------------

This release introduces the capability to automatically skip large files, and
sets default limits on the maximum file size before a file is skipped. Users
should see a performance gain, but may experience warnings associated with
these skipped files.


Upgrades pre 1.0
----------------

* **0.13.x** new rule for quoted literals, option to remove hanging indents in
  rule L003, and introduction of ``ignore_words_regex``.
* **0.12.x** dialect is now mandatory, the ``spark3`` dialect was renamed to
  ``sparksql`` and  datatype capitalisation was extracted from L010 to it's own
  rule L063.
* **0.11.x** rule L030 changed to use ``extended_capitalisation_policy``.
* **0.10.x** removed support for older dbt versions < 0.20 and stopped ``fix``
  attempting to fix unparsable SQL.
* **0.9.x** refinement of the Simple API, dbt 1.0.0 compatibility,
  and the official SQLFluff Docker image.
* **0.8.x** an improvement to the performance of the parser, a rebuild of the
  Jinja Templater, and a progress bar for the CLI.
* **0.7.x** extracted the dbt templater to a separate plugin and removed the
  ``exasol_fs`` dialect (now merged in with the main ``exasol``).
* **0.6.x** introduced parallel processing, which necessitated a big re-write
  of several innards.
* **0.5.x** introduced some breaking changes to the API.
* **0.4.x** dropped python 3.5, added the dbt templater, source mapping and
  also introduced the python API.
* **0.3.x** drops support for python 2.7 and 3.4, and also reworks the
  handling of indentation linting in a potentially not backward
  compatible way.
* **0.2.x** added templating support and a big restructure of rules
  and changed how users might interact with SQLFluff on templated code.
* **0.1.x** involved a major re-write of the parser, completely changing
  the behaviour of the tool with respect to complex parsing.
