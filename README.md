Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the
[Finance minister of Algeria](https://www.wikidata.org/wiki/Q54217583)
was listed an 'instance of: finance minister' rather than subclass,
didn't know what country it was connected to, and wasn't linked in
either direction to the 
[Ministry of Finance](https://www.wikidata.org/wiki/Q3315378), so I
fixed those up.

Step 2: Tracking page
=====================

PositionHolderHistory page created at https://www.wikidata.org/w/index.php?title=Talk:Q54217583&oldid=1233709410

Current status 9 dated memberships, and 8 dated; 10 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit the [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Scrape
==============

Comparison/source = [Ministère des Finances (Algérie)](https://fr.wikipedia.org/wiki/Ministère_des_Finances_(Algérie))

      bundle exec ruby scraper.rb "https://fr.wikipedia.org/wiki/Minist%C3%A8re_des_Finances_(Alg%C3%A9rie)" | tee wikipedia.csv

This was relatively easy to get working (although urlencoding the URL
isn't working because of the bracket)

Step 5: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

11 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/9619948ce5d94/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

24 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/d31b41edcad55
but then it failed on a pre-existing qualifier (Q401173/P580). It looks
like that one was added this morning, but Step 4 *should* have picked
that up, I think. I'm not sure why it didn't unless there was a *lot* of
QueryService lag.

Step 8: Refresh the Tracking Page
=================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q54217583&oldid=1233751726

This still quite a few issues, lots of which are because the French
Wikipedia source doesn't have links for lots of people, and thus we
couldn't auto-resolve their Wikidata IDs.

This is still far from complete, but it's a lot further on from where it
was.
