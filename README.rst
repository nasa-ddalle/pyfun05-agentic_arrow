
.. _pyfun-ex05-agentic-arrow:

Demo 5: Agentic FUN3D on Arrow with OpenCode
============================================

This demo is a variation of the first pyfun example, in which FUN3D is run on
an arrow shape with four fins. The difference here is that none of the work
was done by hand. I opened OpenCode, gave it a simple prompt (the task
described in ``ANALYSIS.md``), and told it to run.

Ok, so there is a little bit more to the tutorial than that! First of all,
you will need an OpenAI-compatible endpoint to be used to power the model. This
can be a locally served model using `llama.cpp <https://llama-cpp.com/>`_,
`vLLM <https://vllm.ai>`_ or other tools. It can also be a cloud-based endpoint
like `https://chatgpt.com/api/v1 <https://chatgpt.com/api/v1>`_. Except you
should not use a public enterprise endpoint like the main Chat GPT because
FUN3D is subject to Export Administration Regulations. Many institutions,
including NASA, have approved services that provide API endpoints to both
Claude and ChatGPT, in addition to models by other major providers.

The setup files are the same as in the first example: the arrow geometry, a
template namelist, and a master JSON file defining a four-case run matrix over
Mach number and angle of attack. There are only slight differences from the
`pyfun01-arrow <https://github.com/nasa-ddalle/pyfun01-arrow>`_ example. The
most noteworthy differences is that the run matrix is defined in a file rather
than in the JSON.

Secondly, this demo uses `OpenCode <https://opencode.ai/>`_. Installing
OpenCode and gaining access to an approved or internal API endpoint are
prerequisites to this tutorial.

The tutorial will demonstrate how well the fully-autonomous CAPE in OpenCode
approach works with a small model
(`Gemma 4 31B <https://huggingface.co/google/gemma-4-31B-it>`_) and a more
capable one in the GPT 5.1 class.

Step 1: Initializing CAPE
----------------------------

Before opening OpenCode, we, as in the other CAPE tutorials, start by running

    .. code-block:: console

        $ ./copy-files.py
        $ cd work/

Then we run a special command to prepare to use CAPE in fully-autonomous mode.

    .. code-block:: console

        $ cape init-agent

This creates two files: :ref:`AGENTS.md <agents-md>` and ``ANALYSIS.md``.

The contents of ``AGENTS.md`` are quite simple:

    .. code-block:: markdown
        :caption: AGENTS.md
        :name: agents-md

        ## CAPE

        This repository uses CAPE to control one or more CFD run matrices.

        Run `cape data-path AGENTS.md` and read the referenced CAPE agent instructions
        before performing CAPE-related tasks.

        The analysis-specific objective and practices are in `ANALYSIS.md`

    .. figure:: figs/opencode-startup.png
        :width: 6in

        OpenCode at startup in the demo folder

What the agent was asked to do is written down in ``ANALYSIS.md``: check the
run matrix, create a new configuration with its own run matrix and PBS
resource requests, submit all four cases to the queue, and then enter a
monitor-and-extend loop until every case met the convergence criteria.  The
screenshots below show how the two runs went.


Using Gemma 4 31B
-----------------

The first attempt used Gemma 4 31B as the model.  OpenCode began by inspecting
the files and asking permission before running commands.

    .. figure:: figs/opencode10-permission.png
        :width: 6in

        OpenCode asking for permission before proceeding

    .. figure:: figs/opencode15-inspect-help.png
        :width: 6in

        Inspecting the tools and their options

It then worked through the task, submitting the cases to the queue.

    .. figure:: figs/opencode20-submit.png
        :width: 6in

        Submitting the cases

    .. figure:: figs/opencode21-thought.png
        :width: 6in

        The agent thinking through the next step

    .. figure:: figs/opencode25-done-ish.png
        :width: 6in

        Declaring the runs done

    .. figure:: figs/opencode30-extract.png
        :width: 6in

        Extracting the results


Using Advanced Models
---------------------

The second attempt used an advanced frontier model, which planned more
carefully before acting.

    .. figure:: figs/opencode110-plan.png
        :width: 6in

        Planning the work up front

    .. figure:: figs/opencode116-edit.png
        :width: 6in

        Editing the input files

At one point the agent got confused, but it recovered on its own.

    .. figure:: figs/opencode118-confused.png
        :width: 6in

        A moment of confusion

It then analyzed the flow histories to judge convergence.

    .. figure:: figs/opencode120-analyze.png
        :width: 6in

        Analyzing the results

    .. figure:: figs/opencode122-analyze.png
        :width: 6in

        More analysis

    .. figure:: figs/opencode125-extend.png
        :width: 6in

        Extending cases that had not converged

    .. figure:: figs/opencode140-done.png
        :width: 6in

        All cases complete

The end result was the same run matrix completed and analyzed, with
considerably less effort on my part than the manual version of this tutorial.
