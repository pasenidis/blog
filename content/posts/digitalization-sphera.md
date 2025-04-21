+++
date = '2024-02-20T22:50:03+03:00'
draft = false
title = 'Digitalization of a logistics-oriented SME'
+++

---

Nowadays, no business with huge volumes of data survives without tailor-made
software that suits the niche it operates in. With this in mind, a customer
approached me and asked to digitize their operations. This article dives into
the process of developing a whole new IT suite for a European parcel shipping
company that is based in Greece.

<!--more-->

---

## Customer Background

This enterprise has been operating for 4 years in the parcel logistics industry,
their main focus is handling parcel logistics to Europe and Western Asia (mainly
ex-Soviet nations) that is targeted to individuals rather than enterprises,
although they serve this niche too.

The company is a subsidiary of [Sphera Group](https://sphera.express) and brands
itself as **Sphera Express**.

It serves Europe, Russia and Western Asia via truck, or ship depending on the
destination region.

Since their start, they've used no-code solutions such as Notion and Airtable
(and even Excel occassionally) to store customer and parcel data, **which in my
opinion is very outstanding for a business in such a niche**, but this became
unreliable once they reached a huge amount of data & realized they are in need
of custom automations: **from customer communication & parcel tracking, to tax
automation and bookkeeping.**

After a quick analysis, the three main bottlenecks were:

1. Workers were losing a lot of time for tasks that **concern messaging with
   clients**, those could be automated. (even with no-code or low-code)
2. Keeping **track of >1 tonne of cargo every week that are distributed
   globally**, unquestionably laborious given the generic solutions they used.
3. Salesperson that accepts the parcel was editing a **Photoshop template to
   label the parcel, and pasted a QR-code**, dull & time inneficient.

## Online Representation

Their digital presence mainly consists of their website and their social media,
the prior was developed by me, when they launched earlier.

It already had everything they need along with integrations for their partners,
so I moved on.

## Reinventing the wheel

When I tell my colleagues that I developed a software to store customers and
their related shipments in the warehouses, of course they tell me "could've used
a CRM or something like that", but this was not the case. It simply did not suit
them, and it was very end-customer-oriented, which **disrupts the focus from the
parcels - therefore losing it's main point.**

_Don't get me wrong, customers are valued as much as the parcel itself, but
without the parcel there would be no customer._

Getting closer to the point, the software that they are now using for operations
was hard to develop mainly because of the exotic nature of their operations,
each region (sometimes not a nation, but a city or a state) had it's own pricing
formula, and statistically speaking, a new region is being served every fiscal
quarter, but is there any point when I have to add the new formula to the DB?
No, so after trying a lot of ways to avoid this, I found a way for the company
staff to manage this.

I introduced a mini "scripting language" to them, that I had implemented just
for this project. Basically, for each region that they operate, they describe
the pricing via linear & non-linear progression (depends on the region, again)
and store that into a DB.

Of course, this introduces a potential injection attack, but I took my measures
and prevented it via validation.

**This whole formula thing is much simpler than it sounds, one just has to
analyze the pattern.**

Below is a screenshot of the main screen of the solution:
![Screenshot of the app](/images/sphera.png)

## Technical Details

Upon customer's approval, I decided to go serverless: instead of using
traditional Django, FastAPI or Flask-backed apps, I went on with the
recently-renovated Next.JS that was then hosted on Vercel & Supabase for
authentication, Postgres & storage, hence minimizing their IT costs, even
better.

Considering Next.JS supports serverless/lambda functions and Supabase offers
GraphQL, I managed to develop quite a performant & secure solution.

Of course, this combination might not be the perfect solution for everyone, but
in my case, it was really the most optimal way I could've thought of, without
large costs on development and maintainment in the long-term.

One could have gone his own way, self-hosting SQL or using Mongo, writing a
backend in FastAPI or Express, using Angular for the frontend, like most
companies in this industry have been doing.

I ignored the industry standards & approached it differently, put some respect
on the IT guy, let him have a beer on a Friday night instead of fixing MySQL
shards.

## What's Next?

This stack has been operating since late 2023, since then we've had a couple minor bugs - which I had to fix on an early Sunday morning...

Next on the list is the social media automation article, which is on it's way.

Thanks for reading this article, share with your friends using the buttons below!
