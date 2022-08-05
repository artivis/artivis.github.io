---
title: "Word ordering and document adjacency for large loop closure detection in 2D laser maps"
date: 2017-07-01
publishDate: 2020-05-09T22:12:35.023482Z
authors: ["J. Deray", "J. Solà", "J. Andrade-Cetto"]
publication_types: ["2"]
abstract: "We address in this letter the problem of loop closure detection for laser-based simultaneous localization and mapping (SLAM) of very large areas. Consistent with the state of the art, the map is encoded as a graph of poses, and to cope with very large mapping capabilities, loop closures are asserted by comparing the features extracted from a query laser scan against a previously acquired corpus of scan features using a bag-of-words (BoW) scheme. Two contributions are here presented. First, to benefit from the graph topology, feature frequency scores in the BoW are computed not only for each individual scan but also from neighboring scans in the SLAM graph. This has the effect of enforcing neighbor relational information during document matching. Second, a weak geometric check that takes into account feature ordering and occlusions is introduced that substantially improves loop closure detection performance. The two contributions are evaluated both separately and jointly on four common SLAM datasets and are shown to improve the state-of-the-art performance both in terms of precision and recall in most of the cases. Moreover, our current implementation is designed to work at nearly frame rate, allowing loop closure query resolution at nearly 22 Hz for the best case scenario and 2 Hz for the worst case scenario."
featured: false
publication: "*IEEE Robotics and Automation Letters*"
doi: "10.1109/LRA.2017.2657796"
url_pdf: http://www.iri.upc.edu/download/scidoc/1833
---
