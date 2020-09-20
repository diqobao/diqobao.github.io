---
layout: post
title: "Behind Linkedin “People Also Viewed"
author: "Jiahui"
categories: blog
tags: [blog, tech]
image: face-1.jpg
---

# Behind Linkedin’s “People Also Viewed”

I seldom use LinkedIn, but every time I click into some one’s Linkedin profile, several familiar names would show up in this “People Also Viewed” section. It is an excellent feature, for connecting you with other people you might be interested in, through a basic recommendation system. However, just like all the noisy posts constantly occupying my timeline on LinkedIn, “People Also Viewed” doesn’t really fit my need to see some eye-catching people that I would love connect to, and instead, it keeps pushing someone I’ve already known.

## Guess the model behind the black box
Ignoring all the sophisticated technical details behind this simple list of profiles, which have to be a lot considering how many users they have, I tried to propose a speculation on what the model looks like, based on resources I found online and my own experience with this feature.

### Co-occurrences
To begin with, as [Jay wrote on Quora](https://www.quora.com/How-exactly-does-LinkedIn-generate-the-viewers-of-this-profile-also-viewed-list-of-users) in 2011, the logic should be built on top of a simple concept based on **co-occurrence**.  Basically it captures the similarity between user profiles based on common interests. In this case, for example, if you and John are both looking at someone’e LinkedIn profile, it’s intuitive to think you two might have a common interest, and when you ask more people to view, it’s natural to give you another person John likes. Moreover, it’s highly likely that you’re always signaling common interest with hundreds or thousands of people when you are viewing a LinkedIn page, depending on its popularity. 

Thus in general, what _People Also Viewed_ does is to use all the “who views who”, “who works for who” data to create a feature vector for everyone, generate a giant sparse matrix representing the number of common interests between two people, and query it for top n people holding most  number of common interests with a certain person and show them on that person’s profile page.

Here, we have an simplified example of the overall process, that a like of "people also viewed" profiles is generated through 3 single factors from John Doe's data.

![an example]({{ site.github.url }}/assets/img/pav-0.png)

### Factors Influencing Outcomes
The most important thing, however, seems to be the selected features that we would like to grasp in the model. _People Also Viewed_ should essentially use “who views who” information, as its name suggests, but a single metric would not satisfy LinkedIn engineers for sure. 
A list of stuffs that might been taken into consideration are:
- people one viewed
- schools one goes/went to
- companies one works/worked for
- job one applied for
- people one connected with
- groups one is in
- job title
- location
- should be more…

That will give us a vector representing the feature of one user like the following one ( 1 := yes):

| viewed by user A | viewed by user B | viewed by user C | went to university of puppy | worked for Amazon | Seattle | 
|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|
| 1 | 1 | 0 | 1 | 0 | 1 |

_this means this user viewed by A and B but not by C, went to university of puppy, living in Seattle but never worked for Amazon._

1 and 0 in the vector stand for two discrete states of true or false. But it doesn’t  have to be always like this. One weight can be added to this is time: that you views someone’s profile today and that you viewed someone’s profile 3 years ago should be telling a quite different story about your preference, at least the ramification on the recommendation should be at different level of confidence. Thus, if you viewed by someone’s profile 1 years ago, the number in your feature vector will be something like 0.01 in stead of 1. If we select a window of 1 year, here is a graph of relationship between date of an action and its weight (or confident level)


### Measuring Similarity
Since the fundamental logic is about **co-occurrence**, the first distance function fit for this use case to measure the similarity between two users is [L1 distance], which measures the number of different occurrence between two vectors.

### Be Fancier
Considering how tech companies love to deal with anything similar to a recommendation system, I would guess they might try updating their model by leveraging neural networks (aka AI), which is not difficult for well-paid talents working there, as long as the performance trumps the old one. A straightforward performance metric is by counting the number of times you click on the link to profiles given by _People Also Viewed_.

also, the model is used for a lot of pages other than individual profiles: LinkedIn is pushing potential jobs you’re suitable apply, linking one college/company to a bunch of other colleges/companies that seem to be related to the one you’re viewing… 

## Personal Notes
I’m constantly disappointed by recommendation systems which are all over internet, but not so interesting. I keep complaining about how the algorithm of Youtube keep pushing repetitive contents to me, and I find it hard to see anything surprising there, despite an unimaginable complicated system and engineer hours behind it. 
LinkedIn  _People Also Viewed_ here, however, starting from a non-ML mapReduce algorithm, is not designed to be that surprising. and I believe, it works pretty well for its use case. Nevertheless, in the future, I’d like to see more unexpected name from the list, whom I’m not able to induce his/her whole LinkedIn profile from that one-line self introduction.