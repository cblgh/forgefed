# GitPub

Copyright © 2018 GitPub Work Group. [Document License][document-license] and [Software License][software-license] rules apply here.

## Overview

GitPub extends from the server-to-server layer of the [ActivityPub][activitypub] specification. Through this, repositories and branches can signal each other for updates.

GitPub only involves in server-to-server signaling. It does not involve in the underlying git operation or implemtation done by different server.

As compare to [GitPubSub][gitpubsub], this specification focus on signaling inter-repository operations (e.g. forking, pull request). [GitPubSub][gitpubsub], on the other hand, focus on signaling commits details in a repository.

In GitPub, each repository branch is represented by "branch" via a URL on a server. A repository is an alias to the default branch of it. The same set of commits on different server corresponds to different branches. 

Every Branch must have:

* **An `inbox`**: How it get notifications from the world.
* **An `outbox`**: How others pull information about it.

Server should notify the branch owner and / or its subscriber about the notification. But when and how a server should do it is outside the scope of this specification.

So:

* Someone's branch can `POST` its notification to your branch's `inbox` to notify your branch of its updates.
* You can `GET` from an someone's branch to see its latest updates.

### Pull Request

Pull Request, or "Merge Request" in some server, represents a request from a downstream branch for a `git pull` or `git merge` operation on an upstream branch.

In such case, when a user wants his / her branch to be pulled, he / she would send a pull request on the upstream branch web interface. Then:

* The upstream server SHOULD, in the background, get the information and commit of the downstream branch and generates an online URL of the request.
* The upstream SHOULD then `POST` to downstream's `inbox` and notify it of the successfully created **pull request**, along with information (TBD) for subscribing comments.
* The downstream SHOULD `POST` to upstream's `inbox` if it has any update (e.g. new commit).
* The downstream server MAY subscribe updates (e.g. comments, close, re-open) of the pull request on the upstream server.

### Fork

When a user wants to fork a remote upstream branch, he / she would fill in the source repository + branch, along with the target organization / account + repo name, to web interface of his / her server. Then,

* The downstream server SHOULD, in background, clone and fetch the inforamtion of the upstream branch to create local branch.
* The downstream server SHOULD then `POST` to upstream's `inbox` and notify it of the successfully created **fork**, along with information of subscribing it.
* The upstream server MAY subscribe updates of the downstream branch.

[activitypub]: https://www.w3.org/TR/activitypub/
[gitpubsub]: https://www.apache.org/dev/gitpubsub.html
[document-license]: LICENSES/DOCUMENT_LICENSE.md
[software-license]: LICENSES/SOFTWARE_LICENSE.md


## Key Workflows

### Fork

#### Workflow





### Pull Request / Merge Request

#### Workflow

1. Lennon made some awesome changes on his "lennon/hello" repository on Server B. He made
  it available on the "awesome" branch there.

2. Lennon say to Server B, "hey, I want to send a PR to upstream server".

   <!--bill-auger (2018/6/12, msg00127):--> 
   For a git PR, this is very plainly nothing more than:

   ```
   {
   	message-type: 'merge-request' ,
   	source-url: 'https://my-homeserver.net/my-repo.git' ,
   	destination-url: 'https://your-homeserver.net/your-repo.git' ,
   	patch-data: 'a-commit-ish-identifier'
   }
   ```

   the notion of upstream vs downstream plays no role here
   <!-- / -->

3. Server B, "Ho. I remember. It is foobar/hello on Server A, right? If not, please tell me."

4. Lennon, "That is correct. Thanks for asking."

   <!-- bill-auger (2018/6/12, msg00127): There is no reason to assume this request should be to THE one and only "upstream" repo where this one was originally cloned from - nor is there is no reason to assume that this server has kept that historical forking information - a merge request can be proposed to and from any "fork" - upstream/downstream is meaningless in git for example - the 'source-url' and 'destination-url' are very appropriately in the single request as above - this operation can even be specified in reverse for convenience - with the person who has changes visiting the destination website and clicking an "open pull request" button; whereby the destination server redirects the user to his home-server to initiate the transaction; carrying all necessary destination data in hand - only when that user is not signed into his home-server, would the destination need to ask the user to type a URL to their home-server (or simply instruct them to go home to login before attempting that action) -->

5. Server B ask Server A, "Hey pal. What are the branches on your foobar/hello?"

6. Server A shown Server B.

7. Server B ask Lennon, "Alright, here are the branches on foobar/hello on Server A. Which base branch do you want?"

8. Lennon, "Mmm... I'd want to base on master".

   <!-- bill-auger (2018/6/12, msg00127): this could be convenient, but is not strictly necessary and is completely meaningless for some VCSs - some VCS have no concept of branches - even in git-federation - the destination server could easily infer what are the most appropriate destination branches - it is a common practice to declare that all merge- requests are to be made against a single "dev" staging branch - i would recommend that each repo defines a default destination branch against which all remote patches should target - that way, if the patches are in conflict with that branch; it is clearly and correctly the responsibility of the sender torebase them -->

9. Server B might show Lennon the diff. Or not. It'll say, "OK. I'll tell Server A. Stay tuned."

10. Server B tell Server A, "Hello. My user Lennon would want to send you a PR. Here is the URL of the branch that we're talking about."
  <!-- bill-auger (2018/6/12, msg00127): yes - AND: here is the git commit-id to pull from me; or whatever patch-data payload is appropriate for that VCS to be clear though, there is no: "THE Branch" - there is some source metadata or raw data and some destination metadata - even in git, source and destination may (indeed usually) have different branch names - a branch is just an alias for a commit and a commit is just a reference to some patch data - so "here is the URL" is the URL of my source git server where the changes can be pulled - but that is appropriate for git - if sending raw patch text as payload then no source URL is necessary or even meaningful - and then in addition, "here is another URL" that is the destination repo i am targeting.-->

11. Server A, "OK. One moment" It checks the repository and branch, or not.

12. Server A check Ken's setting or its own policy. It found that it should not create the PR right the way. It ask Ken, "Hey, there is an in coming PR and this is the branch's URL." It might also give Ken a preview, or not.

13. Ken, "Seems fine. I agree for you to create a PR here. But do not accept it just yet."

14. Server A tell Server B, "OK. The PR is created here."
    <!-- bill-auger (2018/6/12, msg00127): 11, #12, #13 are white-box implementation-specific concerns - at this point after #10 above, the destination server has all of the necessary information to complete the transaction - only #14 the ACK is important to specify -->

15. Server B tell Lennon, "OK, Here is the PR address. Go see it on Server A. Remember to follow their rules. Enjoy"

16. Now. Lennon and Ken may have email conversation, or talks over Server A. Lennon may also need to create an account on Server A to reply Ken's comment. These does not concern the PR protocol itself.
   <!-- bill-auger (2018/6/12, msg00127): users should never interact directly with foreign servers - nor should users ever need to have an account on any foreign host - that is one of the most important features of federation - users should really not need to run any javascript either or even visit a website - there is no good reason why every interaction could not be accomplished with raw HTTP requests carrying a activity-pub object - i would highly suggest that be a design goal to include GET requests for all data that is necessary for compose all "action" requests. -->
   <!-- That is not to say that users should not visit foreign server nor click buttons on them; but none of those mouse clicks should directly initiate any important action on that foreign server - all interactions on foreign servers should be mediated by the user's home-server in the form of JSON objects - servers should always redirect foreign users back to their home-server to vouch for their authenticity and initiate cross-server actions on their behalf - commenting on issues, clicking a "favorite" button, anything that should be authenticated. -->
   <!-- In this flowery "buddy beer conversation" language, that is the destination server saying to the user's home server: "hey buddy, your home user just clicked an important button on me; that would change my state somehow if one of my users had clicked it - are you willing to vouch for that person's authenticity? if so, please formally send me the corresponding request to do "that thing" signed with your key - i do not react directly to foreign user's mouse clicks" -->
   <!-- So in reality, users are not bound by any rules other than what their home-server enforces - their home-server should decide which actions to actually take on behalf of it's users - then each server decides which requests to honor and which to ignore - those are the only "rules" that are important here - they are really just the facts about the implementation details of what that particular server is willing to do for foreign users -->

17. As requested by Ken, Lennon done several updates on Server B lennon/hello. Server B would tell Server A about these new pushes / remove / rebase. It would tell Server B if Lennon simply gave up and removed the branch. But Lennon persisted in the PR.

   <!-- bill-auger (2018/6/12, msg00127): -->
   OK for this. there would need to be an additional field indicating that this merge request is an appendage onto a known existing one:

   ```
   {
   	message-type: 'merge-request',
   	source-url: 'https://my-homeserver.net/my-repo.git (https://my-homeserver.net/my-repo.git)',
   	destination-url: 'https://your-homeserver.net/your-repo.git',
   	patch-data: 'a-commit-ish-identifier',
   	append-to: 'https://your-homeserver.net/your-repo.git/issues/42'
   }
   ```

   it sounds like this is suggesting that contributor would be able to close the request on the foreign server - it not even necessary to that to happen verbally - it is entirely the prerogative of every project maintainer to manage their own instance in whatever way they see fit - if that contribution was offered freely, then it can be merged or rejected regardless of the contributors approval; because the destination server has already pulled the changes into a staging area - the state of the contributor's home-server is never of any concern of the destination - i would recommend that all projects make it clear that all public contributions via the website must offered without contingency

   <!-- / -->

18. Server A knows about the changes at the PR downstream branch. It might display to Ken.

    <!-- bill-auger (2018/6/12, msg00127): there is that word "downstream" again - there is a critical implementation detail between #17 and #18 here -  unless the raw patch text is sent as payload, then what Server A knows, is only the clone URL and checkout point - in order for Server A to "know" about the actual changes, it must clone and checkout that commit into a staging area in order to display it. -->

19. Finally, Ken agree that the PR is ready. Ken merged it and closed the PR from upstream.

20. Server A tells Server B, "Seems the PR is closed. Will let you know otherwise. I know where to find you anyway."
    <!-- bill-auger (2018/6/12, msg00127): i dont understand this part - how could this possibly be otherwise? Server A knows with absolute certainty that it's user closed the issue or if it remains open - if the contributor is interested to know whether the contribution was actually merged; they will probably need to verify that themselves - git servers could verify that trivially, so maybe there could be a separate message for that; but it is not so straight-forward with other VCSs -->

21. Server B trusts Server A and said nothing back.

22. Server B tells Lennon, "The PR has been merged. Congrat."
