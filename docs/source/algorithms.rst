Algorithms
===========

This section details the three main algorithms; review matching, submission grading, and peer grading;
and the guiding premises of each. The submission and review grading algorithms are described first as
requirements imposed by these algorithms will guide the design of the review matching algorithm.

Submission Grading Algorithm  
----------------------------
    **Parameters:** bonus  

    **Precondition:** each submission has a TA review or at least two peer reviews; each peer have given 
    at least two peer reviews.  

    **Input:** TA reviews, peer reviews.  

    **Output:** grades for each submission.

The submission grading algorithm is in the family of *expectation maximization* (EM) or *message passing 
algorithms.* Peer Pal's current algorithm is a variant of the Vancouver algorithm. It simultaneously tries 
to identify peer qualities (variance), submission clarities (variance), and submission grades (expectation). 
It does so by assuming the peers have equal qualities, doing a quality weighted average of the peer reviews 
to determine submission qualities and grades, measuring the peer quality as the squared error to these 
estimated submission grades, and then repeating with the new peer qualities. When calculating peer qualities, 
that peer’s scores are not directly considered (i.e., that peer is left out of the previous calculation of 
submission grades). Without this detail, the algorithm can produce outcomes where a set of peers has very 
high quality and only their scores are taken into account in the submission grades.

Review Grading Algorithm
------------------------
    **Parameters:** minimum average review grade.

    **Precondition:** each peer has at least one submission that was also reviewed by a TA.

    **Input:** TA reviews, peer reviews.

    **Output:** grades for each peer.

To compute a peer’s review grade on a submission the algorithm computes the Manhattan (L1) distance
between the peer review and the TA review (scores are multi-dimensional when the rubric has more than
two items in it). This distance is subtracted from the maximum grade to get the raw grade. Many peers
reviewed the same submission, when the minimum average review grade parameter is set, these the raw
review grades are normalized so that the grade is at least the minimum average review grade. If a peer has
review grades for multiple submissions, these are averaged to obtain the peer’s final review grade.
If the review assignment has a rubric enabled the TA can manually grade the peer reviews and the
automatic review grading algorithm is disabled.

Review Matching Algorithm
-------------------------
    **Parameters:** peer load, total TA load.

    **Input:** list of TAs, list of peers, list of student-submission pairs
    **Output:** matching of peers and TAs to submissions.

    **Postcondition:** each peer reviews at least one submission that is also reviewed by the TA; no peer is
    assigned to review their own submission.

    **Notes:** it is best to have at least four peer reviews per submission that is not TA reviewed to moderate the
    probability that the submission has fewer than two peer reviews (and fails the preconditions of the
    submission grading algorithm).

A basic step in the matching algorithm is to match *n* peers to *m < n* submissions, the submission list
is copied enough times and then trimmed so that there are exactly n submissions in the list. The peers are
randomly shuffled. The peers are matched to submissions in order. This creates a one to many random
matching that is as balanced as possible, meaning the number of peers reviewing each submission is 
*floor(n/m)* or *ceil(d/m)*.

The algorithm proceeds as follows: First, for total TA load *l*, *l* of the *m* submissions are randomly
selected for TA review. Using the basic matching step, each peer is matched to one of these *l* submissions.
Second, for a peer load of *k* and each peer duplicated *k*-1 times, the *kn - n* duplicate peers are matched
to the remaining *m - l* submissions. Third and finally, the matching is checked for required properties: no
peer is assigned to review their own submission and no peer is assigned to review any submission multiple
times. If this check fails, the matching is discarded and the algorithm restarts from scratch. If a matching
is not found in a sufficient number of tries, the algorithm fails.