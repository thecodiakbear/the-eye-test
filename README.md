# The Eye Test

**An AI verification gate that checks whether a generated output actually matches what was asked for - intent-conformance, not just structural validity.**

Most tests confirm an output is *well-formed*. The Eye Test asks the harder question: is it *right* - does it faithfully match the intent behind it? It's designed to catch the failure mode that matters most as AI moves into pipelines: output that is plausible, well-structured, and wrong.

> This is a project write-up describing the concept and its working implementation. The implementation is private.

---

## The problem

Generative models are fluent and confident, and confidently wrong often enough that treating their output as an answer rather than a draft is expensive. In a single chat you can eyeball the result. But as models get chained into agent pipelines - where one step's output becomes the next step's input - there's no human at each hand-off, so a plausible-but-wrong artifact propagates silently and every downstream step inherits it.

Traditional automated tests don't catch this. They check *structural validity* - is it valid JSON, does it compile, does it match a schema. They can't tell you whether the content is actually correct against the intent. The Eye Test exists to gate on that second, harder question.

## What it is

The Eye Test is a verification step that scores a generated artifact - a piece of text, a generated image - against the **intent** behind it: the brief, the source, the thing it was supposed to be. It returns a pass/fail verdict with a written reason, so the result is both actionable and auditable.

Crucially, it's **intent-conformance, not regression.** It doesn't compare against a prior baseline ("did this change from last time"). It asks "does this match what we actually wanted this time" - which is the right question for generative output that's different every run.

## How it works

An independent AI step receives the generated artifact plus its intent (the brief / source / spec), and scores it against a set of defined criteria. For a drafted post that's faithfulness to the source, on-topic, on-voice, within limits. For a generated image it's subject match, legible text, absence of visual artifacts, on-brand style.

The output is a structured verdict - pass/fail per criterion, an overall verdict, and a one-line reason - that downstream automation can route on and a human can read at a glance.

## The key insight: critical vs soft criteria

The make-or-break design decision is *calibration*. A verification gate that's too strict fails everything and gets switched off; one that's too loose lets broken output through. The Eye Test solves this by splitting criteria into **critical** and **soft**:

- **Critical** criteria are the genuine failures - wrong subject, unfaithful claim, unreadable text, visible distortion. Any critical failure fails the artifact.
- **Soft** criteria are nuance - a slightly-off stylistic choice - which are noted but don't block.

This split is what keeps the false-fail rate low enough that the gate stays trusted and switched on. Getting it wrong in either direction is the single most common way a verification layer dies in practice.

## Where it runs

The Eye Test is implemented as the verification gate inside [Distil](../../../distil), a multi-stage content pipeline - sitting between drafting and publishing, checking each drafted post against its source before it reaches human review. That makes it both a working component and a proof of the concept: a live example of AI output being independently verified mid-pipeline rather than trusted.

## Status

Working as the text-verification gate in Distil, with an image-verification design specified. The broader idea - a standalone, metered verification layer for generative pipelines - is an open direction; the concept is proven in a live pipeline.

## Why it matters

As AI produces more of the work, the bottleneck shifts from "can the model make something" to "can we tell, reliably and at scale, when what it made is good versus slop." Verification - grounding output, checking it against intent, catching it when it's confidently wrong - is where a lot of the real value in applied AI now sits. The Eye Test is a concrete attempt at that gate.

---

*Built by Michael Williamson - The Codiak Bear — technical founder and AI engineer. I design agentic AI systems on a direct-and-verify philosophy. (https://www.linkedin.com/in/michael-williamson-1789a774)*
