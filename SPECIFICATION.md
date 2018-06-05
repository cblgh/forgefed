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

[activitypub]: https://www.w3.org/TR/activitypub/
[gitpubsub]: https://www.apache.org/dev/gitpubsub.html
[document-license]: LICENSES/DOCUMENT_LICENSE.md
[software-license]: LICENSES/SOFTWARE_LICENSE.md
