# alpine-ruby

Ruby + Alpine for Rocket ACI conversion.

    FROM nowk/alpine-base:3.2


Includes:

* bundler


| Stats             |         |
| ----------------- | ------- |
| Docker Image Size | ~62 MB  |
| Rocket ACI Size   | ~23 MB  |

---

`ENV` variables

    RUBY_MAJOR        2.2
    RUBY_VERSION      2.2.2
    BUNDLER_VERSION   1.10.5
    GEM_HOME          /usr/local/bundle
    BUNDLE_APP_CONFIG $GEM_HOME
    PATH              $GEM_HOME/bin:$PATH

---

__Converting:__

    docker2aci docker://nowk/alpine-ruby:2.2.2

*Latest version of the actool will properly export the LABEL directives defined 
in the Dockerfile, else please read below.*

Because the `arch` label is not exported, we will need to add that in manually 
by extracting, modifying the manifest then rebuilding the ACI before adding to
our image store.

    tar xvf nowk-alpine-ruby-2.2.2.aci -C alpine-ruby

Add the `arch` label.

    ...
    "labels": [
        ...
        {
            "name": "arch",
            "value": "amd64"
        },
        ...
    ],
    ...

Rebuild the ACI.

    actool build --overwrite alpine-ruby alpine-ruby-2.2.2-linux-amd64.aci

Add to the image store via `rkt fetch`.

    sudo rkt --insecure-skip-verify fetch alpine-ruby-2.2.2-linux-amd64.aci

__Add as a dependency:__

In your app's ACI `manifest`.

    ...
    "dependencies": [
        {
            "imageName": "nowk/alpine-ruby",
            "labels": [
                {
                    "name": "version",
                    "value": "2.2.2",
                },
                {
                    "name": "os",
                    "value": "linux",
                },
                {
                    "name": "arch",
                    "value": "amd64",
                }
            ]
        }
    ],
    ...

__Ruby binary:__

The ruby binary is located at:

    /usr/bin/ruby

