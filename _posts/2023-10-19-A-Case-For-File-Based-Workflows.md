---
title: A Case for Paper Based Workflows
date: 2023-10-19 18:00:00
categories: Programming
tags: [programming,mpanje]
---

A common thread I find in most software developers' product pitches in Malawi (Africa
as a whole too to some extent) revolves around replacing paper based workflows with
almost one hundred percent digital workflows. Complete elimination of paper in any
workflow is seen as a big plus. Whilst I do understand where this line of thinking is
coming from, I find that more often than not these products don't fit the Malawian
environment quite right. Almost every single Malawian has ever found himself/herself
unable to access some public service because there is "no network" or "no electricity."
If you go back to a time when these public services were run on fully manual paper based
workflows, you will hardly find cases where this sort of thing happened. To some extent
I think it's arguable that these public services have overall regressed despite the
efficiency gains from the digitization.

Back in 2018, I joined Baobab Health Trust. I was placed on the team that was responsible
for an application called ART. ART was/is an application that is used in a number (200+)
of clinics in Malawi to deliver HIV/AIDS Antiretroviral treatment. The first iteration of
ART was developed cha m'ma thu sauzandemu. There are many lessons I learnt when I was
working on ART. Of all the lessons, one particularly stands out for me. I was made to
always think of situations when ART fails. Should clinics come to a complete hault because
of a bug in the application? Or perhaps there is no electricity and the servers are off,
should the clinic stop running? The obvious answer here is no; the clinic must keep
running one way or the other. ART was built from the ground up to work alongside a paper
based workflow. ART enhances the paper based workflow instead of eliminating it.
In situations where ART fails, clinics are still able to operate (albeit
not efficiently) following the paper based workflow.

## How exactly does ART enhance a paper based workflow?

When using ART, clinics are required to maintain a physical file for every patient.
The physical file contains a good amount of the patient's medical history that's enough
for a medical practioner to use in treating the patient. So, when a patient comes to a
clinic and is searched for in ART, the application provides the file number of the
patient's physical file. It also provides an additional optional feature that helps in
locating where exactly that file is located in the shelves where files are stored.

An ART workflow is broken down into multiple stages. At each stage ART prints out some
small labels providing a summary of all the information collected and generated at that
stage. The labels are then pasted on a page in the physical file. Essentially, a copy of
part of the data in ART is made available in physical form. The data in the physical file
is enough for a clinician to use in attending to a patient during ART down times.
For example, when a patient is prescribed some medication, a label having the prescription
is printed out. The prescription label is then pasted in the patient's file. The next time
the patient comes and ART is not available, information about the patient's last
prescription is available to the clinician to use if needed. As you can see, ART has not
eliminated something the clinician already does but has rather enhanced it. In the absence
of ART, the clinician still has to write that prescription down in the file. ART just
makes that process easier. ART is simply making the clinician more efficient at his/her
work, not completely eliminating it.

To keep things in sync with the physical files after an ART outage, ART provides a Back
Data Entry (BDE) mode that allows the information that clinicians wrote down in the files
to be entered into the application. BDE operates with somewhat looser validations and
restrictions. The reasoning behind this lowering of restrictions is that if shit has
already gone down, there is no point in correcting it at that point. If a patient has
been given unsuitable medication, that incident needs to be kept track of, as it is now
the patient's history. At the time I left all this ART business in 2021, there were talks
of introducing Optical Character Recognition (OCR) for BDE. I don't know how far the ART
team has gone with this but we can all agree this is awesome stuff.

## What's the point I am trying to make?

As developers we must realise that we are developing applications for low resource
environments. It's not out of the ordinary to have 18 hours of no electricity in Malawi.
Almost every week we get an SMS from one of our ISPs that fibre waduka ku Mazambiki.
In addition to all of this, we must be aware that we will never stop developing buggy
applications (even with this AI powered programming). Develop applications with a mindset
that they will fail and have a back up plan for when that happens. In my opinion, I believe
physical files are one of the best tools we have available for our back up plans. Embrace
file based workflows... I die a little every time I fail to access some service because
palibe network. A service that can technically be provided bhobho without a computer is
blocked chifukwa network palibe? Pathetic, really... This is a problem that some Malawians
solved cha m'ma 2005 mu.

> Of course outsourcing development of the applications used in
the provision of these services does not help at all. Local developers have more context
on Malawi's challenges than anthu akunja.
{: .prompt-info}
