
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

This creates two files: ``AGENTS.md`` and ``ANALYSIS.md``. The contents of
``AGENTS.md`` are quite simple:

    .. code-block:: markdown
        :caption: AGENTS.md

        ## CAPE

        This repository uses CAPE to control one or more CFD run matrices.

        Run `cape data-path AGENTS.md` and read the referenced CAPE agent instructions
        before performing CAPE-related tasks.

        The analysis-specific objective and practices are in `ANALYSIS.md`

This ``AGENTS.md`` points the model and harness to the longer instructions that
apply to all solver modules in CAPE and describe how to use CAPE to run CFD
(and how to discover more documentation). For users who already have an
``AGENTS.md`` file in their run repo, this section will be appended (unless
there's already a Markdown session entitled "CAPE"). ``AGENTS.md`` is a fairly
standard file name that most harnesses will recognize in some way, possibly
even telling the model to read it by default. ``ANALYSIS.md``, however, not an
industry-wide standard, and so the ``AGENTS.md`` file calls it out.

After running ``cape init-agent``, you will see a template file that looks like
this:

    .. code-block:: md

        # Analysis objective

        Describe what this analysis is intended to determine.

        # Procedure

        Describe anything special about the run procedure you want the agent to follow.
        The agent will already know how to do basic CAPE procedures, but if you have
        any special instructions for handling errors, what settings can be altered,
        etc., place them here.

        # Completion criteria

        Describe what constitutes a successful analysis that can be approved. Leaving
        this section blank by design will leave the decision open to the model you are
        serving, provided it has vision capability.

        Examples:
        - Converge to a steady state up to 15000 iterations for every case
        - Accept a limit cycle oscillation with steady or shrinking amplitude for cases
        with unsteady/time-accurate inputs **only**; do not go beyond 20,000
        iterations.
        - Accept the recommendations of `cape get-case-state` up to a hard limit of
        30,000 iterations.
        - Accept the recommendations of `cape get-col-state` for each reported
        subfigure; approve cases that are close after 20,000 iterations and use
        30,000 iterations as a hard cutoff.

This obviously has template information and is the file where CFD subject
matter experts can customize how CAPE behaves. The **Completion criteria** in
particular may be different from one solver and run matrix to another.

An example filled-out ``ANALYSIS.md`` file is present in the repo. You can just
copy it into the ``work/`` folder ... or add your own comments to it.


Using Gemma 4 31B
-----------------

The first attempt used Gemma 4 31B as the model. Below is a screenshot of the
prompt tried (which was eventually successful).

    .. figure:: figs/opencode-startup.png
        :width: 6in

        OpenCode at startup in the demo folder

What the agent was asked to do is written down in ``ANALYSIS.md``: check the
run matrix, create a new configuration with its own run matrix and PBS resource
requests, submit all four cases to the queue, and then enter a loop of
monitoring and extending or approving until every case met the convergence
criteria.  The screenshots below show how the two runs went.

Gemma began by inspecting the files and asking permission before running
commands. Because the CAPE primary ``AGENTS.md`` file is located outside the
run repo, OpenCode tells Gemma to ask for permission first.

    .. figure:: figs/opencode10-permission.png
        :width: 6in

        OpenCode asking for permission before proceeding

Gemma then went about following the procedure. The example procedure outlined
in ``ANALYSIS.md`` is purposefully very tentative so that models might be able
to complete part of it even if the overall procedure is too challenging for the
model to complete in one shot. It called ``cape --help`` (despite the
``AGENTS.md`` saying to run ``cape -h``, interestingly) to learn more about the
CAPE CLI.

    .. figure:: figs/opencode15-inspect-help.png
        :width: 6in

        Inspecting the tools and their options

Without performing too much analysis, it started submitting cases to the queue.
After submitting the cases, it uses the ``cape wait`` command (as instructed)
to wait for 2 cases (``-n 2``) to require action. This command prevents the
model from staying active while the CFD is running normally.

    .. figure:: figs/opencode20-submit.png
        :width: 6in

        Submitting the cases

Expanding the *Thinking* block just before the ``cape wait`` command, you can
see Gemma's internal reasoning. It's not too sophisticated, but it gets the job
done in this case.

    .. figure:: figs/opencode21-thought.png
        :width: 6in

        The agent thinking through the next step

Without much fanfare, after the cases completed (meaning their status was
``DONE`` when ``cape wait`` ended), Gemma simply approved each case. It
generated the PDF report along the way but failed to make good use of it. (This
will likely be better by the time you read this tutorial as CAPE makes viewing
the subfigures easier for small-ish models.)

    .. figure:: figs/opencode25-done-ish.png
        :width: 6in

        Declaring the runs done

In addition, Gemma forgot the final step (``cape extract``), so there was a
second prompt reminding it to do so.

    .. figure:: figs/opencode30-extract.png
        :width: 6in

        Extracting the results


Using Advanced Models
---------------------

The second attempt used a more advanced model, though still far from the 2026
frontier, which planned more carefully before acting. It even created a *Todo*
list, which is one of the tools made available by OpenCode.

    .. figure:: figs/opencode110-plan.png
        :width: 6in

        Planning the work up front

The instructions include bits about forking the template ``pyFun.json`` file
and making edits, which more advanced models handle with *ease*. And OpenCode
prevents a nice visual presentation of these edits (though you might have to be
paying close attention to even notice it).

    .. figure:: figs/opencode116-edit.png
        :width: 6in

        Editing the input files

The agent also picked up on minor instructions in CAPE's primary ``AGENTS.md``
about committing changes to ``git`` if possible. Gemma did not process that
instruction at all; and here you can see a more advanced model working through
the fact that it's in a sandbox folder such that the files are not tracked by
``git``.

    .. figure:: figs/opencode118-confused.png
        :width: 6in

        A moment of confusion--and resolution

After following the other instructions, including submitting the cases and
running ``cape wait``, it utilized several methods to assess convergence,
including CAPE's built-in heuristic from ``cape get-col-state``.

    .. figure:: figs/opencode120-analyze.png
        :width: 6in

        Analyzing the results

The ``cape get-subfig`` command is the best option for agents to actually
*look* at plots in a fashion very similar to human-in-the-loop analysis. At the
time this sample was collected ``cape get-subfig`` did not convert the PDFs to
raster images (required for most vision-enabled Large Language Models), so the
agent did that on its own with a call to ``pdftoppm``. You can then see the
reasoning after viewing the result. It seems very reasonable, but models
presumably don't get exhausted if they have to do this for 100s of cases.

    .. figure:: figs/opencode122-analyze.png
        :width: 6in

        More analysis

Unlike Gemma, this model found reason to extend some of the cases, which it did
and then reentered the ``cape wait`` loop.

    .. figure:: figs/opencode125-extend.png
        :width: 6in

        Extending cases that had not converged

Eventually, the agent declared all cases complete. Interestingly, not every
case was run the same number of iterations. This was intended! The idea is that
CAPE can minimize CPU/GPU resources while still ensuring each case meets the
pre-declared convergence criteria. And, of course, this time the model didn't
forget to run the post-processing step.

    .. figure:: figs/opencode140-done.png
        :width: 6in

        All cases complete

