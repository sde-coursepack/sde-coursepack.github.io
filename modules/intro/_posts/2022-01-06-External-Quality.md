---
Title: External Software Quality
---

![A man calling 911 because a toaster stabbed him in the face. The 911 operator asks him if he read the manual. When the caller says no, the operator hangs up on him.](https://imgs.xkcd.com/comics/rtfm.png)    
Source: [XKCD #293](https://xkcd.com/293/) by Randall Munroe

# External Software Quality
{: .no_toc }


*External software quality* refers to the quality of the software from
the perspective of the **stakeholders**. In this module, we will discuss who the Stakeholders are, and what are the components of external software quality.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---

 

## Who are Stakeholders?

*Stakeholders* are anyone affected
by the software, not just the user; that is, anyone who "holds a stake"
in the usage of the software.

To explain the distinction between *stakeholders* and *users*, consider
an electronic medical records system at a large hospital. Who are
the stakeholders?

* Doctors and nurses - they fill out and maintain medical records.
  They will also want to view medical history, monitor medication, view
  changes in conditions over time, etc.
* Resource Managers - they will need to track medical expenditures,
  medication supplies, how many beds are available in which departments,
  whether a hospital has the resources to treat a patient with a given condition.
* IT Employees - If something goes wrong with the medical records or someone
  is unsure how to perform a specific task with the electronic medical records,
  they are the primary support staff.
* Insurance Managers - will look at treatments to determine coverage.
* Patients - While they are unlikely to use the system directly, they are
  directly affected by it. A data corruption or incorrect usage by a doctor or nurse due to a confusing interface could result in incorrect medication or treatment.
* The hospital board - While they will likely never use the system directly,
  the board of the hospital is probably going to make the purchasing decision
  on any large-scale software for the department. While they will likely
  work with their staff to select the best system, the board is going
  to have to weigh factors that the doctors, nurses, and IT staff may not,
  like price, warranties, insurance, etc.

All of these stakeholders have different priorities of goals! So, we
want to define external software quality broadly enough to encompass
this.

With this in mind, let's consider the following **external** quality measures:
* Functionality
* Reliability
* Usability
* Efficiency
* Portability

## Functionality

*Is the software functionally complete? That is, does it do
everything it is supposed to do?*

Functionality is about to what extent the system meets the customer's
needs in what the software *does*. Does the software have all the
features the customers need or expect? Is the software sufficiently
secure? Are the results the software produces accurate?

## Reliability

*What is the capability of the software to maintain performance
under certain conditions over a certain period of time?*

Consider Amazon Web Services (AWS): Many web applications rely on AWS,
like Airbnb, Slack, Lyft, Netflix, Yelp, etc. If Amazon Web Services
had a significant outage, these applications would all stop working. However, it also runs Amazon's own
distribution and shipping infrastructure. If your website, application,
or technology rely on AWS, what could go wrong? In an AWS outage last year,
smart appliances like Refrigerators, Door Locks, Robot Vacuum Cleaners,
Self-Cleaning Litter Boxes, etc. stopped working. Imagine how
this could affect, say, a customer that doesn't carry their physical
keys with them because they use their phone to lock and unlock their door?


## Usability

*How much effort is needed for a customer to use software?*

How much learning is required to interact with software? Do the
layouts, aesthetics, and colors help guide the user's vision effectively?

In Collab, instructors can add a Web Link to the tabs on their course page,
like we have on our Collab for the Coursepack and Course Drive Folder.
What if I need to edit the Web Link? Well, if I go to Site Settings,
I Can See tabs for:

* Site Information
* Edit Site Information
* Manage Tools
* Tool Order
* Add Participants
  etc.

Which of those would you go to change the URL of a Web Link? The obvious
answer is "Manage Tools". The obvious answer is also wrong. If I want
to change the URL of a Web Link, I have to go to "Tool Order"!

![A screenshot of Collab showing the only way to edit the URL of a Web Link is under "Tool Order"](../images/collab_usability.png)

The first time I used Collab, I honestly thought this feature didn't exist.
I spent months where, if I needed to change a URL, I would use "Manage Tools"
to delete the existing Web Link and create another, which was far less efficient.

Why do I use a Google Drive Folder for class resources instead of
the Resources tool in Collab? Because it's far more efficient.

These are *usability* issues, that make the customer experience of using the
software worse, thus lowering the overall software quality.

Accessibility is also a factor in usability: that is, can the software
be used by people with a broad range of characteristics.

## Efficiency

*When operating, what resources are used, and to what extent, by the software?*

*Resources* above typically refers to time and memory, but can also reference
other resources. For example, if a program is using an internet connection,
how much bandwidth does it use? What are the necessary speeds for the program
to operate? Generally, the fewer resources a program consumes, the better.


## Portability

*How able is the software to be transferred from one environment to another?*

Can your software run on Windows, Mac, Linux, etc.? Does your app
work in Chrome, Firefox, Edge, etc.? Can elements within the software
adapt to a change in how and where the users uses the software?