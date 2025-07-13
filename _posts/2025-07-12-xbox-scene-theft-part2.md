---
layout: post
comments: false
status: publish
published: true
title: Theft in the Xbox Scene - Part 2
description: This is not an article I want to be writing. It is not something I ever imagined I would have to write.
permalink: /blog/xbox-scene-theft-part-2/
author: Dustin
date: '2025-07-12 09:32:45 -0400'
date_gmt: '2025-07-12 14:32:45 -0400'
categories:
- Blog
image: xbox-scene-theft/header.png
thumb: generic.png
---

# Introduction

This is a continuation of Theft in the Xbox Scene - Part 1, where I began documenting some of the challenges and setbacks
we’ve faced as a result of ongoing IP theft and misinformation.

<!--more-->

## Unresolve Issues
There’s been a lot of confusion about the core issue raised in the original post. To be clear, it’s the **theft of our
software**. That was the first and most important part of the article to be written. Everything else was added to provide
background and context; but unfortunately, that also introduced a lot of noise into the conversation.

Despite the attention the post received, very little has been done to address these concerns. The key issues remain
unresolved.

### Unresolved
- Theft of XboxHD+ logic
- Theft of HDD unlocking (key recovery) logic
- Theft of CPU upgrade support logic

I also believe the In-Game Reset (IGR) logic copied from rmenhal’s GPL-licensed work deserves more attention within the
community. However, since this isn’t my code and we don’t use it, we’re not pursuing it ourselves.

### Partially Addressed
PrometheOS has taken responsibility for incorporating XboxHD+ code. However, the issue remains unresolved, as the license
terms are still not being followed. A formal request will be submitted to address this.

![GitHub](/assets/img/blog/xbox-scene-theft-2/1.jpg)

Since PrometheOS is built and distributed using Cerbios, we also believe the underlying Cerbios-related issues need to be
addressed as part of that resolution.

### DMCA
Over the past year, we have made multiple attempts to address the unauthorized distribution of XboxHD+ files on xbins.
This has included outreach from others in the retro community, but so far, those efforts have not led to a resolution.

A DMCA notice was submitted to GitHub for Pandora, a tool built specifically to download files from xbins. Most of these
files are not legally available through any other means. To clarify, our concern lies only with the distribution of our
works without permission. If members of the Cerbios and PrometheOS teams have contacts at xbins, we hope they will step
in to help resolve the issue.

We have also submitted DMCA notices to online stores selling products that include or enable unauthorized use of our protected
works. This includes hardware specifically designed to circumvent the XboxHD+, such as boards marketed with phrases like
“just find the firmware,” as well as, in one case, an OpenXenium product explicitly sold with Cerbios and PrometheOS preloaded.

In the case of Cerbios and PrometheOS listings, we have stepped back from issuing further notices for now. We hope the
teams behind those projects will take responsibility and work to resolve the issue themselves instead of leaving the
burden entirely on third-party sellers.

# Theft of HDD Unlocking (Key Recovery) Logic

There has been a lot of confusion surrounding what was stolen and why it matters. To clarify: drive unlocking and drive
unlocking via key recovery are two very different processes.

Drive unlocking through vendor commands is often a fallback method for manufacturers. It allows access to drives
that are locked but without recovering the original key. In contrast, drive unlocking via key recovery uses a diagnostic
interface to run custom payloads that extracts the drive’s unique key directly from its controller.

The Seagate U5 is a perfect example of why key recovery is not straightforward. There is no predefined command to ask
the drive for its key. Unlocking it requires a custom payload, executed through an undocumented diagnostic interface, to
force the controller to execute code fromo a specific location in memory that places the hardware in just the right state
for the key to remain accessible and not cleared from memory.

I reached out to EqUiNoX for clarification on how their implementation works but have not received a clear response.

The initial list of resources they shared includes:
- Documents covering TTY-based unlocking with and without key recovery
- References to drives with completely different architectures
- One closed-source program, which I have not been able to confirme to support key recovery on original Xbox drives.

## The Elephant in the Room
A common challenge with publishing technical write-ups is having to respond to made-up accusations. I want to address
one of them directly because it containts a direct quote from EqUiNoX of Team Resurgent.

On July 8th I was tagged in a post on Bluesky claiming that one of the many samples we provided was a direct copy of
Microsoft code.

![BlueSky](/assets/img/blog/xbox-scene-theft-2/2.jpg)

Not only is this incorrect, but the cropped screenshot being circulated is misleading. It omits surrounding context that
differentiates our implementation from Microsoft's. This kind of selective framing only fuels misinformation.

I addressed this on the OGXbox forums.

Below is the full section in question without the crop. I've added labels to make it clear which code is Project Stellar,
PrometheOS, and the same retail Xbox kernel. This is not "leaked source code" or anything like that. It's the exact same
kernel as the one in the BlueSky post. It's the ``Retail 3944`` kernel with the SHA1SUM of
``67054fc88bda94e33e86f1b19be60efec0724fb6`` for the full 1MB backup.

![Ghidra](/assets/img/blog/xbox-scene-theft-2/3.jpg)

This is from the **same retail kernel** as the cropped screenshot and was decompiled using Ghidra with default settings.
Kernel exports were renamed using ``RenameKernelImports.py``, generated from the XboxDev wiki infromation on the kernel
exports, and type information was applied from ``nxdk``.

The original code in question is a polling routine used to ensure the drive resets. It does not match Project Stellar. We
included it in our list of samples because it exists outside of the kernel and the pattern lifted from Project Stellar
into PrometheOS is still clear. The timings also still matter. If the surrounding logic changes, the delays could change
too or be optimized away if that was the goal.

To be absolutely clear, I am are not claiming ownership over basic I/O instructions like ``out`` or standard functions like
``KeStallExecutionProcessor()``. That would be like claiming ownership over ``if()`` statements.

Our implementation is based on the ATA/ATAPI specification, as shown below from the ``Information Technology - AT Attachment
with Packet Interface - 6 (ATA/ATAPI-6)`` datasheet.

![Datasheet](/assets/img/blog/xbox-scene-theft-2/4.jpg)

The timing values used in our implementation apply a multiplier to account for bus delays and timer resolution differences.
These timers are based on the onboard crystal, and ownership of the bus from the CPU through the northbrige and to the
southbridge must be queued. While the timings are not exact, our values were tested extensively to ensure stability.

But you might ask, “The PrometheOS code uses a variable to store ``mIdePort``. Isn’t that obviously different from either
implementation?”

![Ghidra](/assets/img/blog/xbox-scene-theft-2/5.jpg)

And to that I would say this shows a basic misunderstanding of how the Xbox works. If this had actually been reverse
engineered from the Xbox kernel, that variable would not exist. The Xbox has only a single IDE controller, and it is
always mapped to a fixed I/O address. Anyone doing even basic reverse engineering would know this. In fact, just looking
at the motherboard would make it clear that it only has one IDE controller because it only has one IDE port.

If your concern is “optimizations,” then that’s a free one. Because as it stands, ``mIdePort``, does nothing but make
it one line of code different from Stellar.

## Team Resurgent's Response

![GitHub](/assets/img/blog/xbox-scene-theft-2/6.jpg)

After publishing the first article, I opened a GitHub issue on the PrometheOS repository to seek clarification on the
HDD unlocking logic. The goal was to ask direct, technical questions about the origins of the code and the development
process behind it.

The responses from EqUiNoX were appreciated, and I want to be clear that I gave every opportunity for clarification. I
asked specific questions, followed up where needed, and allowed their words to speak for themselves.

However, several of the responses raised new questions, revealed inconsistencies, or introduced claims that will need to
be addressed in more detail. For now, I want to provide a summary of what was said and what is still unclear. I will
expand on these concerns later in the article.

## Response #1

![GitHub](/assets/img/blog/xbox-scene-theft-2/7.jpg)

I find the statement ``hddVscUnlocker::resetIdeBus <- derived from the retail kernel decompilation reset bus delays``
questionable. PrometheOS requires Cerbios to execute from a modchip, and both projects have worked together publicly in
the past. This is important context, because Cerbios is compiled from leaked Xbox source code.

In the implementation of ``resetIdeBus``, the oddity of using ``mIdePort`` has already been addressed above. As for the
claims regarding how the remaining timing values were derived, there would need to be actual testing to validate that assertion.

The most significant red flag is the statement about ``Sleep()`` that it causes
``issues with background threads accessing hard drive``, and that this is why ``KeStallExecutionProcessor()`` was chosen.
This is simply not how the Xbox works. The Xbox uses [preemptive multitasking](https://en.wikipedia.org/wiki/Preemption_(computing)),
meaning it constantly and rapidly switches between threads.

To prevent other threads from accessing the drive, the correct behavior is to raise the interrupt request level (IRQL)
to ``DISPATCH_LEVEL`` or higher to block thread switching via ``KeRaiseIrqlToDpcLevel``.

![Microsoft API](/assets/img/blog/xbox-scene-theft-2/8.jpg)

![Microsoft API](/assets/img/blog/xbox-scene-theft-2/9.jpg)

But PrometheOS doesn't do this. Instead, it copies Stellar’s exact logic: raising the IRQL to the interrupt level of the
IDE controller by calling ``HalGetInterruptVector`` and ``KfRaiseIrql``. And this isn't the only way to achieve this.

![Ghidra](/assets/img/blog/xbox-scene-theft-2/10.jpg)
<p style="text-align:center">The only difference here being that IRQL is raised by the caller via callee. ie PrometheOS
moved the KeRaiseIrqlToDpcLevel call from outside the drive functions to right inside them.</p>
<br />

This is not for blocking other threads. It's for blocking kernel responses to IDE controller interrupts.

![Quote](/assets/img/blog/xbox-scene-theft-2/11.jpg)

This undermines the explanation that it was done for optimization. I don't belive this is a misunderstanding on my part.
It reads more like an excuse layered over copied logic.

---

To drive this home... Something I noticed while putting this article together, and honestly found a bit funny, is this
[line](https://github.com/Team-Resurgent/PrometheOS-Firmware/blob/main/PrometheOSXbe/PrometheOSXbe/hddVscUnlocker.cpp#L522)
in the PrometheOS 1.3.0 source release:

```HalGetInterruptVector(0xe, &irql); // 0xe = PNPISABus```

The value ``0xe`` is the same value as the interrupt for the IDE controller, but the comment is completely incorrect.
``PNPISABus`` is a constant from the ``INTERFACE_TYPE`` enum used to describe how devices identify themselves to the system,
such as PCI or ISA. It is not related to interrupt levels in this context.

![Microsoft API](/assets/img/blog/xbox-scene-theft-2/12.jpg)

This is without a doubt a case of someone not understanding what the code is doing. The comment was likely added to make
sense of a value they copied without knowing what it actually represented. It brings everything full circle.

## Response #2

![Quote](/assets/img/blog/xbox-scene-theft-2/13.jpg)


### Lack of Source Code
```Unfortunately no, when working on a new PrometheOS version I tend to work locally and when ready to push, push as one commit...```

This development approach is incompatible with open-source license requirements, especially for any project that uses
GPL-licensed code. The GPL requires that the complete source code be made available for **all** distributed versions.
That includes any intermediary versions that were released as binaries, such as v1.2.0 of PrometheOS. Failing to
retain or provide this source code is a direct violation of the license.

This is not a small or technical oversight. It is a license breach that affects the legality of distributing the software
itself. If you distribute a binary, you are also required to provide the full corresponding source to that binary. There
is no exception for working locally or pushing code in one large commit. By not maintaining source control in a transparent
and trackable way, the project undermines the trust that open-source licensing is built on.

What makes the situation worse is that PrometheOS includes code taken from XboxHD+, which is also released under the GPL.
When that code is reused, modified, or redistributed, the GPL still applies. The license terms are not optional or negotiable.

PrometheOS is distributing binaries that contain our GPL-licensed code, yet it fails to:
- Provide the complete source code for the versions being distributed
- Properly attribute the original work
- Grant the same rights to others to inspect, modify, and redistribute under the same terms

This is not theoretical. These are the exact obligations outlined in GPLv2, sections 1 and 3.

If this were simply a misunderstanding, it could have been addressed by releasing the required source for v1.2.0. Instead,
the jump from v1.2.0 to v1.3.0 includes binary changes that appear deliberately obfuscated, making it harder to identify
what was changed and why. The only changelog entry provided was “Improved: VSC HDD Unlocking Times,” yet the code changes
go far beyond timing adjustments and raise serious questions about intent.

### Sleep() and KeStallExecutionProcessor()
```I had trouble with Sleep() because it doesn't halt background threads from accessing the drive whilst unlocking is
taking place. KeStallExecutionProcessor() will halt any background threads, along with raising the IRQL.```

This is not true and is covered above. ``KeStallExecutionProcessor()`` has no impact on running threads and does not read,
write, or modifiy the IRQ level in any way.

### Research Notes
```I do remember decompiling a dos utility called zu.exe however which helped greatly.```

Earlier in this article, I covered the topic of HDD Unlocking (Key Recovery) and the confusion surrounding the type of
proof being requested. That context is important here, because the list of "prior research" provided in defense of the
PrometheOS implementation largely references TTY-based unlocking methods or closed-source software that either does not
support key recovery or does not apply to the drives in question.

One new reference introduced in this discussion was ``zu.exe``. I had not encountered this tool before, so I asked for
clarification on exactly which drives it can unlock and whether the screenshot provided was showing key recovery in action.

These details matter because the architecture of these drives varies significantly, and claiming to have "extrapolated"
key recovery from closed software, without documentation or code, raises serious questions.

![Capture](/assets/img/blog/xbox-scene-theft-2/14.jpg)

![Capture](/assets/img/blog/xbox-scene-theft-2/15.jpg)

Still, I tested the theory in good faith. I used an Intel P4 system and a motherboard built with the exact same chipset
used in the Xbox, but was unable to reproduce the result.

## Response #3...

![Capture](/assets/img/blog/xbox-scene-theft-2/16.jpg)

This was one of the last meaningful replies in the GitHub conversation, and it left several questions unanswered.

# Obfuscations in the v1.3.0 Source Release

PrometheOS is currently hiding behind the 1.2.0 source release, but after comparing the 1.2.0 binary to the 1.3.0 release
over the past few days, I've identified even more undeniable examples of copied code. In this case, the changes appear to
have been made specifically to conceal the theft.

The release notes for version 1.3.0 only mention one related change:

``Improved: VSC HDD Unlocking Times``

With that in mind, we will focus on examples where the code was modified in ways that are unrelated to unlocking
performance or any reasonable optimization. These changes do not improve execution speed or stability. They appear to
serve only one purpose: to make the stolen code harder to recognize.

## Meaningless Changes (Obfuscations)

![Capture](/assets/img/blog/xbox-scene-theft-2/17.jpg)

We pressed hard for clarity on how the Seagate U5 was figured out, because as far as we know, there is no public
documentation or tool that covers key recovery for it.

In comparing the 1.2.0 and 1.3.0 versions of PrometheOS, two changes stood out:

``sgVscDiagExecCmd(..., "GFFF3")`` was changed to ``sgVscDiagExecCmd(..., "GFFFFFFF3")``
``sgVscDiagExecCmd(..., "/1")`` was changed to ``sgVscDiagExecCmd(..., "/2")``

These changes have no meaningful impact. In fact, the first one arguably makes things worse.

The value ``GFFF3`` instructs the hard drive controller to begin code execution at 0xFFF3. Extending it to ``GFFFFFFF3``
results in the same truncated address due to the controller's address space, but it adds unnecessary characters that make
the command slower to transmit and process.

The change from ``/1`` to ``/2`` just switches diagnostic input modes, but neither option has an effect on the controller's
behavior in this flow.

These are not optimizations. They are changes that serve no functional purpose and appear to exist only to obscure the
original source of the code.

# Conclusion
At this point, I’ve said everything I need to. The concerns have been laid out clearly, and the patterns speak for
themselves. Unless something meaningfully changes or new information comes to light, this will be my final word on the
matter.
