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

#### A. Source Server Initiation

1. A user sees a repository (e.g. https://server-a.net/user-a/bar) on server A in the federated network.
2. The page has a "Fork" button, or any other represetation. He or she initiates the fork from the origin server by clicking on a fork button there.
3. The user clicks the "fork" button on the repo.
4. Server A asks the user for his / her home server.
5. The user filled in the server URL (e.g. https://server-b.net).
6. Server A make a request to Server B to:
  1. Make sure that it supports federation;
  2. Find the forking endpoint to initiate the forking (e.g. https://server-b.net/fork)
7. Server A redirects the user to the forking endpoint with the URL of the repository to fork.
  (e.g. https://server-b.net/fork?repo=https%3A%2F%2Fserver-a.net%2Fuser-a%2Fbar)

#### B. Forking

1. The user either:
  1.  specify the repository to fork with endpoint parameter:
    (e.g. https://server-b.net/fork?repo=https%3A%2F%2Fserver-a.net%2Fuser-a%2Fbar), or
  2. specify the repository to fork manually:
2. Server B resolves the clone-URI from Server A using to-be-defined means:
  (e.g. webfinger https://server-a.net/.well-known/webfinger?res=server-a.net%2Drepo)
3. Server B clones the repo from Server A locally (the server applies security means here as required).
4. A new repository on Server B is created with its own URL
  (e.g. https://server-b.net/user-b/bar)

#### C. Forking Mention

If Server B is polite, it POST a notification to the inbox of source server (e.g. https://server-a.net/user-a/bar/inbox):

```json
{
  "message-type": "fork",
  "source-url": "https://server-a.net/user-a/bar",
  "destination-url": "https://server-b.net/user-b/bar",
  "timestamp": "2018-06-06T12:21:32.000Z"
}
```

#### D. Subscriptions between Repository

A repository might be interested in displaying information considering another repository. For example,

1. Source repository wants to display updates from all fork destinations; or

2. Destintion repository wants to display updates of the source repository.

then the repository can always subscribes to updates of the other.

Say if the repository https://server-a.net/user-a/bar subscribes to the fork destination https://server-b.net/user-b/bar. If the user B pushed to https://server-b.net/user-b/bar, it will POST an activity to the inbox of subscribers, includes https://server-a.net/user-a/bar/inbox:

```json
{
  "message-type": "push",
  "url": "https://server-b.net/user-b/bar",
  "vcs-type": "git",
  "vcs-url": "https://server-b.net/user-b/bar.git",
  "vcs-spec": "master",
  "vcs-data": "some-commit-hash",
  "timestamp": "2018-06-08T13:24:12.000Z"
}
```

Server A could then update the fork's history state. (E.g. by `git fetch`).

### Pull Request / Merge Request

#### A. Creating a Pull Request / Merge Request with Suggestions

1. Initiation
  1. Lennon made some awesome changes on his "lennon/hello" repository on Server B. He made
    it available on the "awesome" branch there.
  2. Lennon say to Server B, "hey, I want to send a PR somewhere".
2. Repository Specification with Suggestion
   1. Server B, "Ho. I remember your current source code is from "foobar/hello" on Server A. Do you want to send the PR there? If you have another repository in mind, please tell me".
   2. Lennon, "That is correct. Thanks for asking." He confirmed that the repository is foobar/hello on server A.
3. Branch / PR Target Specification with Suggestion
   1. Server B ask Server A, "Hey pal. What are the available targets for a PR?"
   2. Server A shown Server B a list of branch names, or whatever suitable for the VCS.
      Server A may specify one of them as the "default".
   3. Server B ask Lennon, "Alright, here are the available PR targets of foobar/hello on Server A.
     Which do you want to send the PR to?"
   4. Lennon, "Mmm... I'd want to base on master".
4. Preview
   1. With the specified Repository and PR Target, Server B may use underlying VCS protocol (e.g. git pull + diff) to show Lennon the diff.
   2. Lennon, "Mmm... Seems nice." I confirm this is what I want to do.
   3. Server B, "OK. I'll tell Server A. Stay tuned."

#### B. Making the Pull Request / Merge Request

1. Server B tell Server A, "Hello. My user Lennon would want to send you a PR. Here is the URL of the branch that we're talking about."

    ```json
    {
      "message-type": "awesome",
      "source-url": "https://server-b.net/lennon/hello.git",
      "source-spec": "awesome",
      "source-data": "git-commit-hash-or-vcs-identifier",
      "destination-url": "https://server-a.net/foobar/hello.git",
      "destination-spec": "master"
    }
    ```

2. Server A immediate replied, "OK. One moment".

    ```json
    {
      "status": "pending",
      "request": {
        "message-type": "awesome",
        "source-url": "https://server-b.net/lennon/hello.git",
        "source-spec": "awesome",
        "source-data": "git-commit-hash-or-vcs-identifier",
        "destination-url": "https://server-a.net/foobar/hello.git",
        "destination-spec": "master"
      }
    }
    ```

#### C. Formally Considering the Pull

1. Outside the scope of this specification*, Server A wants to be sure that Ken want this. So,

  1. Server A may checks the source repository and branch.
  2. Server A may check Ken's setting or its own policy. It found that it should not create the PR right the way. It ask Ken, "Hey, there is an in coming PR and this is the branch's URL." It might also give Ken a preview, or not.
  3. Ken, "Seems fine. I'll consider this PR"

2. Server A creates an entinty "Pull Consideration / Merge Consideration". Server A tell Server B, "OK. We are considering the PR here."

    ```json
    {
      "message-type": "awesome",
      "url": "https://server-a.net/foobar/hello/pull/123",
      "source-url": "https://server-b.net/lennon/hello.git",
      "source-spec": "awesome",
      "source-data": "git-commit-hash-or-vcs-identifier",
      "destination-url": "https://server-a.net/foobar/hello.git",
      "destination-spec": "master"
    }
    ```

3. Server B should tell Lennon on their UI, "OK, Here is the PR address. Go see it on Server A. Remember to follow their rules. Enjoy"

#### D. Communicating About the Pull

1. *Outside the scope of this specification*, Ken and Lennon might communicate on how to pushes / rebase / stash or do whatever to make the PR appropriate.

  Lennon and Ken may have email conversation, or talks over Server A. Lennon may also need to create an account on Server A to reply Ken's comment.

2. As requested by Ken, Lennon done several updates on Server B lennon/hello. Server B would tell Server A about these new pushes / rebase / stash. It would tell Server A if Lennon simply gave up and removed the branch. But Lennon persisted in the PR.

    ```json
    {
      "message-type": "awesome",
      "source-url": "https://my-homeserver.net/my-repo.git",
      "destination-url": "https://your-homeserver.net/your-repo.git",
      "patch-data": "a-commit-ish-identifier",
      "append-to": "https://your-homeserver.net/your-repo.git/issues/42"
    }
    ```

3. Server A knows how to get updated repository status from the notice. It may display to Ken. If needed, it may use the VCS-specific mechanism (e.g. git checkout) to fetch the latest source code for comparison.


#### E. Merging / Pulling

1. Finally, Ken agree that the PR is ready. Ken merged it and closed the PR from upstream.

2. Server A tells Server B, "Seems the PR is closed. Will let you know otherwise. I know where to find you anyway."

    ```json
    {
      "message-type": "merge",
      "source-url": "https://my-homeserver.net/my-repo.git",
      "destination-url": "https://your-homeserver.net/your-repo.git",
      "append-to": "https://your-homeserver.net/your-repo.git/issues/42"
    }
    ```

3. Server B trusts Server A and said nothing back.

4. Server B tells Lennon, "The PR has been merged. Congrat."
